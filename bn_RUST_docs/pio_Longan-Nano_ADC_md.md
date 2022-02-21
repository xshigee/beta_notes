
2020/5/18+

PlatformIO Longan Nano ADC
# PlatformIO Longan Nano ADC

## 概要
Longan-NanoでADCを使う(framework:gd32v版)。
(ホストPCとしてはubuntuを想定している)


## PlatformIOのインストール
```bash

python3 -m venv pio_env
source pio_env/bin/activate

pip3 install platformio

インストール後も、本ツールを使用する場合
同じディレクトリで以下を実行する：  
source pio_env/bin/activate
# 「source」は、「.」でも良い

```
## 準備
以下を実行して、udevのrulesを登録する：
```bash


curl -fsSL https://raw.githubusercontent.com/platformio/platformio-core/master/scripts/99-platformio-udev.rules | sudo tee /etc/udev/rules.d/99-platformio-udev.rules

sudo udevadm control --reload-rules

sudo usermod -a -G dialout $USER
sudo usermod -a -G plugdev $USER

```

## テスト用プロジェクト sample-adc を作成/実行する
```bash

#ターゲットボードのtarget名を検索する
# (ここではlonganを検索する)

pio boards | grep -i logan
#出力例：
sipeed-longan-nano       GD32VF103CBT6  108MHz       128KB    32KB   Sipeed Longan Nano
sipeed-longan-nano-lite  GD32VF103C8T6  108MHz       64KB     20KB   Sipeed Longan Nano Lite

# flash容量が２種類あるが、持っているボードのものを選択する
#　128kBのほうを持っているので、target名としては「sipeed-longan-nano」になる。

# プロジェクト sample-adc のディレクトリを作成する
#(サンプルgd32v_lcdを流用する)

git clone https://github.com/sipeed/Longan_GD32VF_examples.git
cd Longan_GD32VF_examples

cp -r gd32v_lcd sample-adc
cd sample-adc

nano platformio.ini
# 以下の内容になるように編集する：
```

```

;PlatformIO Project Configuration File
;
;   Build options: build flags, source filter
;   Upload options: custom upload port, speed and extra flags
;   Library options: dependencies, extra library storages
;   Advanced options: extra scripting
;
; Please visit documentation for the other options and examples
; https://docs.platformio.org/page/projectconf.html

[env:sipeed-longan-nano]
platform = gd32v
board = sipeed-longan-nano
framework = gd32vf103-sdk

upload_protocol = dfu
debug_tool = sipeed-rv-debugger

#upload_protocol = um232h
#debug_tool = um232h

#upload_protocol = sipeed-rv-debugger
#debug_tool = sipeed-rv-debugger

```
「upload_protocol = 」、「debug_tool = 」は、自分の環境に合わせること。

続き：
```bash
# テスト用のmain.cを作成する
nano src/main.c
次の内容に編集(差し替え)する：
```
```c

#include "gd32vf103.h"
#include "systick.h"
#include <stdio.h>
//#include "gd32vf103v_eval.h"

#include "lcd/lcd.h"
#include "fatfs/tf_card.h"
#include <string.h>

uint16_t a0;
uint16_t a1;
uint16_t a2;
uint16_t a3;
uint16_t a4;
uint16_t a5;
uint16_t a6;
uint16_t a7;
uint16_t b0;
uint16_t b1;


void init_uart0(void)
{	
    /* enable GPIO clock */
    rcu_periph_clock_enable(RCU_GPIOA);
    /* enable USART clock */
    rcu_periph_clock_enable(RCU_USART0);

    /* connect port to USARTx_Tx */
    gpio_init(GPIOA, GPIO_MODE_AF_PP, GPIO_OSPEED_50MHZ, GPIO_PIN_9);
    /* connect port to USARTx_Rx */
    gpio_init(GPIOA, GPIO_MODE_IN_FLOATING, GPIO_OSPEED_50MHZ, GPIO_PIN_10);

    /* USART configure */
    usart_deinit(USART0);
    usart_baudrate_set(USART0, 115200U);
    usart_word_length_set(USART0, USART_WL_8BIT);
    usart_stop_bit_set(USART0, USART_STB_1BIT);
    usart_parity_config(USART0, USART_PM_NONE);
    usart_hardware_flow_rts_config(USART0, USART_RTS_DISABLE);
    usart_hardware_flow_cts_config(USART0, USART_CTS_DISABLE);
    usart_receive_config(USART0, USART_RECEIVE_ENABLE);
    usart_transmit_config(USART0, USART_TRANSMIT_ENABLE);
    usart_enable(USART0);

    usart_interrupt_enable(USART0, USART_INT_RBNE);
}

int _put_char(int ch)
{
    usart_data_transmit(USART0, (uint8_t) ch );
    while ( usart_flag_get(USART0, USART_FLAG_TBE)== RESET){
    }

    return ch;
}

void adc_init(void) {
    rcu_periph_clock_enable(RCU_GPIOA);
    gpio_init(GPIOA, GPIO_MODE_AIN, GPIO_OSPEED_50MHZ, GPIO_PIN_0);

    RCU_CFG0 |= (0b10 << 14) | (1 << 28); // ADC clock = 108MHz / 8 = 13.5MHz(14MHz max.)
    rcu_periph_clock_enable(RCU_ADC0);
    ADC_CTL1(ADC0) |= ADC_CTL1_ADCON;
}


uint16_t get_adc(int ch) {
    ADC_RSQ2(ADC0) = 0;
    ADC_RSQ2(ADC0) = ch;
    ADC_CTL1(ADC0) |= ADC_CTL1_ADCON;

    while( !(ADC_STAT(ADC0) & ADC_STAT_EOC) );

    uint16_t ret = ADC_RDATA(ADC0) & 0xFFFF;
    ADC_STAT(ADC0) &= ~ADC_STAT_EOC;
    return ret;
}


int main(void)
{
    unsigned char s[256] = {'\0'};
    
    /* system clocks configuration */
//    rcu_config();

    /* ADC configuration */
    adc_init();

    init_uart0();

    Lcd_Init();         // init OLED
    LCD_Clear(BLACK);
    BACK_COLOR=BLACK;

    while(1){
        a0 = get_adc(0);
        a1 = get_adc(1);
        a2 = get_adc(2);
        a3 = get_adc(3);
        a4 = get_adc(4);
        a5 = get_adc(5);
        a6 = get_adc(6);
        a7 = get_adc(7);
        b0 = get_adc(8);
        b1 = get_adc(9);

        // value print
        printf(" A0: %d\r\n", a0);
        printf(" A1: %d\r\n", a1);
        printf(" A2: %d\r\n", a2);
        printf(" A3: %d\r\n", a3);
        printf(" A4: %d\r\n", a4);
        printf(" A5: %d\r\n", a5);
        printf(" A6: %d\r\n", a6);
        printf(" A7: %d\r\n", a7);
        printf(" B0: %d\r\n", b0);
        printf(" B1: %d\r\n", b1);
        printf("-------------- \r\n\r\n");

        sprintf(s, "A0: %d         ", a0);
        LCD_ShowString(0,  0, &s, WHITE);
        float v0 = a0 * 3.3 / 4095.0;
        int f0 = (int)v0;
        int f1 = (int)((v0-f0)*100);
        sprintf(s, "A0(V): %d.%02d         ", f0,f1);
        LCD_ShowString(0,  16, &s, GREEN);
        sprintf(s, "A3: %d         ", a3);
        LCD_ShowString(0,  32, &s, BRED);
        sprintf(s, "A4: %d         ", a4);
        LCD_ShowString(0,  48, &s, GBLUE);
        sprintf(s, "A6: %d         ", a6);
        LCD_ShowString(0,  64, &s, RED);


    }
}
```
続き：
```bash

# build
pio run


# ボードをPCに接続して
# 以下の手順で書き込みモード(DFUモード)にする：
# (1)「BOOT」ボタンと「RESET」ボタンも同時に押す。
# (2)１秒くらいして「RESET」ボタンを離す。
# (3)その後、「BOOT」も離す。

# build&upload(flash)
pio run -t upload
# buildしないで書き込む場合は以下を実行する：
pio run -t nobuild -t upload -v
# -v は、詳細を表示するオプション


# 以上で、基本的な操作としては完了となる

```

書き込み後、LCDにADCの値(PA0,PA3,PA4,PA6)が表示される。
シリアル出力にもADCの値(PA0からPA7,PB0,PB1)が出力される。   
ただし、以下のポートは、他の機能端子と兼用になっているので
独立して使用できるのは、A0,PA3,PA4,PA6になる。

| port | func |  
| ---: | :--- |   
| PA1 | LED_G |  
| PA2 | LED_B |  
| PA5 | TFT_SCL|  
| PA7 | TFT_SDA |  
| PB0 | TFT_RS |  
| PB1 | TFT_RST |  

 


## 参考情報

[Longan NanoでRISC-Vチャレンジ(7) - ADC/DAC](http://blueeyes.sakura.ne.jp/2019/10/26/2681/)  
[Longan Nano(Schematic).pdf](https://dl.sipeed.com/LONGAN/Nano/HDK/Longan%20Nano%202663/Longan%20nano%202663(Schematic).pdf)  
[GD32VF103_Firmware_Library_User_Guide_V1.0.pdf](http://www.gd32mcu.com/data/documents/yingyongbiji/GD32VF103_Firmware_Library_User_Guide_V1.0.pdf)  

[Ｓｉｐｅｅｄ　Ｌｏｎｇａｎ　Ｎａｎｏ　ＲＩＳＣ－Ｖ　ＧＤ３２ＶＦ１０３ＣＢＴ６開発ボード](http://akizukidenshi.com/catalog/g/gK-14678/)   
■主な仕様  
・CPU：GD32VF103CBT6  
・メモリ：128KB Flash/32KB SRAM  
・160×80ドット、0.96インチのフルカラーIPS液晶  
・TFスロット(microSDスロット)  

[Longan Nano PINOUT](https://longan.sipeed.com/assets/longan_nano_pinout_v1.1.0_w5676_h4000_large.png)    
[download Longan Nano Datasheet](https://cn.dl.sipeed.com/LONGAN/Nano/Spec/Sipeed%20longan%20nano%20Datasheet%20V1.0.pdf)


[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

[Longan nano で Hello World!](https://qiita.com/myston/items/d7366021f75dce000c3b)   
[Sipeed Longan Nanoで文字を表示してみる](http://kyoro205.blog.fc2.com/blog-entry-667.html)   
・図形描画の関数の説明がある  
[Sipeed Longan Nano 用の、FONTX2対応LCD描画ライブラリとサンプルプログラム](https://github.com/takamame0205/LonganNanoLCD)  


以上

