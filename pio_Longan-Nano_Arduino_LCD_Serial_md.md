
2020/5/19

PlatformIO Longan Nano LCD Serial
# PlatformIO Longan Nano LCD Serial

## 概要
Longan-NanoでLCDとSerialを使う(framework:Arduino版)。
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

## テスト用プロジェクト arduino-lcd-serial を作成/実行する
```bash

#ターゲットボードのtarget名を検索する
# (ここではlonganを検索する)

pio boards | grep -i logan
#出力例：
sipeed-longan-nano       GD32VF103CBT6  108MHz       128KB    32KB   Sipeed Longan Nano
sipeed-longan-nano-lite  GD32VF103C8T6  108MHz       64KB     20KB   Sipeed Longan Nano Lite

# flash容量が２種類あるが、持っているボードのものを選択する
#　128kBのほうを持っているので、target名としては「sipeed-longan-nano」になる。

# プロジェクト arduino-lcd-serial のディレクトリを作成する
#(サンプルmicrosを流用する)

git clone https://github.com/smuehlst/longan-nano-experiments.git
cd longan-nano-experiments
cp -r micros arduino-lcd-serial
cd arduino-lcd-serial

nano platformio.ini
# 以下の内容になるように編集する：
```

```

; PlatformIO Project Configuration File
;
;   Build options: build flags, source filter, extra scripting
;   Upload options: custom port, speed and extra flags
;   Library options: dependencies, extra library storages
;
; Please visit documentation for the other options and examples
; http://docs.platformio.org/page/projectconf.html

[env:sipeed-longan-nano]
platform = gd32v
framework = arduino
board = sipeed-longan-nano
monitor_speed = 115200
platform_packages = framework-arduino-gd32v @ https://github.com/sipeed/Longduino.git

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
nano src/main.cpp
次の内容に編集(差し替え)する：
```
```cpp


#include <Arduino.h>
extern "C" {
#include "lcd/lcd.h"
}
#include "gd32vf103.h"
#include <stdio.h>
#include <inttypes.h>

static void longan_oled_init(void)
{
    Lcd_Init();
    LCD_Clear(BLACK);
    BACK_COLOR = BLACK;
}

static void serial_init(void)
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

}

/* retarget the C library printf function to the USART */
int _put_char(int ch)
{
    usart_data_transmit(USART0, (uint8_t) ch );
    while ( usart_flag_get(USART0, USART_FLAG_TBE)== RESET){
    }

    return ch;
}
//===========================================

int a0;
float a1;
int a2;
int a3;
int a4;
int a5;

void setup()
{
  longan_oled_init();
  serial_init();

  // initialize LED digital pin as an output.
  pinMode(LED_BUILTIN, OUTPUT); // LED_R
  pinMode(PA1, OUTPUT); // LED_G
  pinMode(PA2, OUTPUT); // LED_B

  a0 = 1234;
  a1 = 45.78;
  a2 = 910;
  a3 = 2345;
  a4 = 6789;

  printf("serial start...\n\r");
}

void loop()
{

  char s[256] = {'\0'};  // buffer for sprintf

  // display lcd
  sprintf(s, "A0: %d         ", a0);
  LCD_ShowString(0,  0, (u8 const *)s, WHITE);
  int f0 = (int)a1;
  int f1 = (int)((a1-f0)*100);
  sprintf(s, "A1: %d.%02d         ", f0,f1);
  LCD_ShowString(0,  16, (u8 const *)s, GREEN);
  sprintf(s, "A2: %d         ", a2);
  LCD_ShowString(0,  32, (u8 const *)s, BRED);
  sprintf(s, "A3: %d         ", a3);
  LCD_ShowString(0,  48, (u8 const *)s, GBLUE);
  sprintf(s, "A4: %d         ", a4);
  LCD_ShowString(0,  64, (u8 const *)s, RED);

  // value print serial
  printf(" A0: %d\r\n", a0);
  printf(" A1: %d.%02d\r\n", f0, f1);
  printf(" A2: %d\r\n", a2);
  printf(" A3: %d\r\n", a3);
  printf(" A4: %d\r\n", a4);
  printf("-------------- \r\n\r\n");

  a0 += 1;
  a1 += 0.01;
  a2 += 1;
  a3 += 1;
  a4 += 1;

  // LED Red
  digitalWrite(LED_BUILTIN, LOW);
  digitalWrite(PA1, HIGH);
  digitalWrite(PA2, HIGH);
  delay(200);
  // LED Green
  digitalWrite(LED_BUILTIN, HIGH);
  digitalWrite(PA1, LOW);
  digitalWrite(PA2, HIGH);
  delay(200);
  // LED Blue
  digitalWrite(LED_BUILTIN, HIGH);
  digitalWrite(PA1, HIGH);
  digitalWrite(PA2, LOW);
  delay(200);

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
書き込み後、LCDに変数の値(a0,a1,a2,a3,a4)が表示される。
シリアル出力に変数の値(a0,a1,a2,a3,a4)が出力される。   
LEDは、赤、緑、青の順で繰り返し色が変化する。
 
## 補足
[TODO list #1](https://github.com/sipeed/Longduino/issues/1)を見るとanalogReadの実装は終わっているようだが、実際にanalogRead(0)などを動かしてみると-1を値として返すようだ。(なにかしらのupdateが上手く行っていない？？)    

以下に確かにソースは存在している。。。     
https://github.com/sipeed/Longduino/blob/master/cores/arduino/wiring_analog.c

## 参考情報

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

