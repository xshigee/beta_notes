
2020/4/30

PlatformIO cli Longan Nano
# PlatformIO cli Longan Nano

## 概要
開発ツールPlatformIOをcli(comand line interface)で使う(Longan-Nano版)。VisualCodeのプラグインとしてPlatformIOを使用することができるが、ここでは、cliとしての使い方について記する。
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

## テスト用プロジェクト sample-longan を作成/実行する
```bash

#ターゲットボードのtarget名を検索する
# (ここではlonganを検索する)

pio boards | grep -i logan
#出力例：
sipeed-longan-nano       GD32VF103CBT6  108MHz       128KB    32KB   Sipeed Longan Nano
sipeed-longan-nano-lite  GD32VF103C8T6  108MHz       64KB     20KB   Sipeed Longan Nano Lite

# flash容量が２種類あるが、持っているボードのものを選択する
#　128kBのほうを持っているので、target名としては「sipeed-longan-nano」になる。

# プロジェクト sample-longan のディレクトリを作成する
mkdir sample-longan	
cd sample-longan
# 以下を実行して必要なファイルを作成する
pio init --board sipeed-longan-nano

nano platformio.ini
# arduinoとして動かすので、以下の内容になるように編集する：
```

```

; PlatformIO Project Configuration File
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
#framework = gd32vf103-sdk
framework = arduino
upload_protocol = dfu
```
続き：
```bash
# テスト用のmain.cppを作成する
nano src/main.cpp
次の内容に編集する：
```
```c++

/*
 * Blink
 * Turns on an LED on for one second,
 * then off for one second, repeatedly.
 */

#include <Arduino.h>

// Set LED_BUILTIN if it is not defined by Arduino framework
// #define LED_BUILTIN 2

void setup()
{
  // initialize LED digital pin as an output.
  pinMode(LED_BUILTIN, OUTPUT);
}

void loop()
{
  // turn the LED on (HIGH is the voltage level)
  digitalWrite(LED_BUILTIN, HIGH);
  // wait for a second
  delay(1000);
  // turn the LED off by making the voltage LOW
  digitalWrite(LED_BUILTIN, LOW);
  // wait for a second
  delay(1000);
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
書き込み後、LEDが点滅すれば動作としてはＯＫとなる。

## sample#1
sample#1は、arduinoではなくgd32vf103-sdkによるLED点滅プログラムになる：

```bash
git clone https://github.com/sipeed/platform-gd32v.git
cd platform-gd32v/examples/longan-nano-blink

# platformio.iniの内容の確認：
# 以下のようであればOK
```
platformio.ini 
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
framework = gd32vf103-sdk
board = sipeed-longan-nano
monitor_speed = 115200

upload_protocol = dfu

```
続き：
```bash

# ボードをPCに接続して
# 以下の手順で書き込みモード(DFUモード)にする：
# (1)「BOOT」ボタンと「RESET」ボタンも同時に押す。
# (2)１秒くらいして「RESET」ボタンを離す。
# (3)その後、「BOOT」も離す。
pio run -t upload

# 以上で書き込みが終わり、LEDが点滅する。

```

## sample#2
sample#2は、LCDにアニメーションを表示するデモになる：
```bash

$ cd pio_ws
$ . pio_env/bin/activate

git clone https://github.com/sipeed/Longan_GD32VF_examples.git
cd Longan_GD32VF_examples

#　32GB以下のmicroSDに以下の２つのファイルをコピーして、
# そのmicroSDをボードのTFスロットに刺しておく。
$ ls put_into_tf_card/
bmp.bin  logo.bin

cd Longan_GD32VF_examples/gd32v_lcd

pio run

# ボードをPCに接続して
# 以下の手順で書き込みモード(DFUモード)にする：
# (1)「BOOT」ボタンと「RESET」ボタンも同時に押す。
# (2)１秒くらいして「RESET」ボタンを離す。
# (3)その後、「BOOT」も離す。
pio run -t upload

# 以上で書き込みが終わり、LCDにデモのアニメーションが表示される。

```

## sample#3
sample#3は、helloworld的なプログラムになる：
```bash

# 既存のプロジェクトをコピーしてhelloworld的なプログラムを作る
cp -r gd32v_lcd gd32v_hello

cd gd32v_hello

```
platformio.iniの確認：   
以下のようであればOK
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
debug_tool = sipeed-rv-debugge
```

以下のプログラムを作成する：  
src/main.c
```c

#include "lcd/lcd.h"
#include "fatfs/tf_card.h"
#include <string.h>

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


int main(void)
{

    rcu_periph_clock_enable(RCU_GPIOA);
    rcu_periph_clock_enable(RCU_GPIOC);
    gpio_init(GPIOC, GPIO_MODE_OUT_PP, GPIO_OSPEED_50MHZ, GPIO_PIN_13);
    gpio_init(GPIOA, GPIO_MODE_OUT_PP, GPIO_OSPEED_50MHZ, GPIO_PIN_1|GPIO_PIN_2);

    init_uart0();

    LEDR(1);
    LEDG(1);
    LEDB(1);

    Lcd_Init();         // init OLED
    LCD_Clear(BLACK);
    BACK_COLOR=BLACK;

    LCD_ShowString(0,  0, (u8 *)("Hello World! #0"), WHITE);
    LCD_ShowString(0, 16, (u8 *)("Hello World! #1"), BLUE);
    LCD_ShowString(0, 32, (u8 *)("Hello World! #2"), BRED);
    LCD_ShowString(0, 48, (u8 *)("Hello World! #3"), GBLUE);
    LCD_ShowString(0, 64, (u8 *)("Hello World! #4"), RED);
    Draw_Circle(120,40,25,RED);
    LCD_DrawLine(120,0,130,80,BLUE);
    LCD_DrawRectangle(120,40,159,79,GREEN);
    LCD_Fill(130,20,150,30,MAGENTA);

    printf("printf test start...\n\r");
    int n = 0;
    while (1)
    {
        printf("printf test %d\n\r", (int)n);
        n++;
        LEDR_TOG;
        delay_1ms(200);
        LEDG_TOG;
        delay_1ms(200);
        LEDB_TOG;
        delay_1ms(200);
    }

}
```
続き：
```bash

# ボードをPCに接続して
# 以下の手順で書き込みモード(DFUモード)にする：
# (1)「BOOT」ボタンと「RESET」ボタンも同時に押す。
# (2)１秒くらいして「RESET」ボタンを離す。
# (3)その後、「BOOT」も離す。

pio run -t upload

#　書き込むと、LCDに"Hello World!"の文字列と図形が表示され、LEDが色を変えながら点滅する・
#  USBserialを接続していると以下のような文字列が表示される。
#  (USBserialは、ボードのGND,T0,R0とクロス接続する)


$ picocom /dev/ttyUSB0 -b115200
# /dev/ttyUSB0は、自分の環境に合わせること
printf test start...
printf test 0
printf test 1
printf test 2
printf test 3
printf test 4
printf test 5
printf test 6
printf test 7
....
```


## 参考情報

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

