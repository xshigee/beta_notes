
2020/6/14+:  
・SERIAL_TEST,SERIAL0_ECHO,SERIAL_2CH,AT_TESTを追加した。

2020/6/13+:  
初版

PlatformIO cli Wio Lite RISC-V
# PlatformIO cli Wio Lite RISC-V

## 概要
開発ツールPlatformIOをcli(comand line interface)で使う(Wio_Lite_RISC-V版)。    
VisualCodeのプラグインとしてPlatformIOを使用することができるが、ここでは、cliとしての使い方について記する。
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
過去に設定してあったとしても、rulesが更新されている場合があるので、再設定したほうが良い。


## テスト用プロジェクト wio_sample を作成/実行する
```bash

#ターゲットボードのtarget名を検索する
# (ここではwioを検索する)
pio boards | grep -i wio
#出力例：
seeed_wio_lite_mg126           SAMD21G18A   48MHz        256KB    32KB   Seeeduino Wio Lite MG126
seeed_wio_terminal             SAMD51P19A   120MHz       496KB    192KB  Seeeduino Wio Terminal
wio_link           ESP8266  80MHz        4MB      80KB   Wio Link
wio_node           ESP8266  80MHz        4MB      80KB   Wio Node
wio_lite_risc-v          GD32VF103CBT6  108MHz       128KB    32KB   Wio Lite RISC-V
wio_3g                     STM32F439VI     180MHz       2MB       256KB     Seeed Wio 3G
....

#target名として「wio_lite_risc-v」が判明した

# プロジェクト wio_sample のディレクトリを作成する
mkdir wio_sample	
cd wio_sample
# 以下を実行して必要なファイルを作成する
pio init --board wio_lite_risc-v
pio platform update
```
```bash
# arduinoとして欠落しているファイルがあるのでダウンロードする：
cd ~/.platformio/packages/framework-arduino-gd32v/variants 
mkdir wio_lite_risc-v
cd wio_lite_risc-v/
wget https://raw.githubusercontent.com/sipeed/Longduino/master/variants/wio_lite_risc-v/pins_arduino.c
wget https://raw.githubusercontent.com/sipeed/Longduino/master/variants/wio_lite_risc-v/pins_arduino.h
# 書き込み(dfu-util)ツールのデフォルトバージョンでは、wio_lite_risc-vの書き込みが失敗するので
# 正常に書き込めるバージョンをインストールする：
#(失敗するバージョンでも、表示メッセージは書き込みが正常に終了したように見えるので要注意)
mkdir ~/wio-lite-dfu/
cd wio-lite-dfu

git clone https://git.code.sf.net/p/dfu-util/dfu-util
cd dfu-utils
./autogen.sh
./configure
make

# 以下のように実行ファイルをコピーする：
cp src/dfu-prefix ~/.local/bin/ 
cp src/dfu-suffix ~/.local/bin/ 
cp src/dfu-util ~/.local/bin/ 
cp src/dfu-prefix ~/.platformio/packages/tool-gd32vflash/
cp src/dfu-suffix ~/.platformio/packages/tool-gd32vflash/
cp src/dfu-util ~/.platformio/packages/tool-gd32vflash/

```

```
#元のディレクトリに戻す
cd wio_sample

nano platformio.ini
以下にように編集する：
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

[env:wio_lite_risc-v]
platform = gd32v
board = wio_lite_risc-v
framework = arduino
upload_protocol = dfu
```
続き：
```bash
# テスト用のmain.cppを作成する
nano src/main.cpp
以下のように編集する：
```
```c++


#include <Arduino.h>

#define LED_BUILTIN PA8 // for wio-lite-rv
//#define LED_BUILTIN PC13 // for longan-nano

void setup() {
    // initialize digital pin LED_BUILTIN as an output.
    pinMode(LED_BUILTIN, OUTPUT); 
}

// the loop function runs over and over again forever
void loop() {
    digitalWrite(LED_BUILTIN, HIGH);
    delay(1000);
    digitalWrite(LED_BUILTIN, LOW);
    delay(1000);
}
```
ツールのテストとしてCPUが同じlongan-nanoでも動作テストができるように
LEDのポートを#defineで定義して切り換えられるようにしている。

続き：
```bash

# build
pio run

# ボードをホストPCに接続する
# DFUモードにするために
# (BOOT0スイッチを１にして)resetを１度押す
# build&upload(flash)
pio run -t upload
# buildしないで書き込む場合は以下を実行する：
pio run -t nobuild -t upload -v
# -v は、詳細を表示するオプション

# (書き込み終了後、BOOT0を0にする)
# 以上で、基本的な操作としては完了となる

```
オンボードのLEDが点滅すれば動作としてはＯＫとなる。

出力例：
```

$ pio run -t upload
Processing wio_lite_risc-v (platform: gd32v; board: wio_lite_risc-v; framework: arduino)
---------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/gd32v/wio_lite_risc-v.html
PLATFORM: GigaDevice GD32V 1.1.2 > Wio Lite RISC-V
HARDWARE: GD32VF103CBT6 108MHz, 32KB RAM, 128KB Flash
DEBUG: Current (altera-usb-blaster) External (altera-usb-blaster, gd-link, jlink, rv-link, sipeed-rv-debugger, um232h)
PACKAGES: 
 - framework-arduino-gd32v 0.1.1 
 - tool-gd32vflash 0.1.0 
 - tool-openocd-gd32v 0.1.1 
 - toolchain-gd32v 9.2.0
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 1 compatible libraries
Scanning dependencies...
No dependencies
Building in release mode
Checking size .pio/build/wio_lite_risc-v/firmware.elf
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
RAM:   [=         ]   7.4% (used 2420 bytes from 32768 bytes)
Flash: [=         ]   7.3% (used 9592 bytes from 131072 bytes)
Configuring upload protocol...
AVAILABLE: altera-usb-blaster, dfu, gd-link, jlink, rv-link, serial, sipeed-rv-debugger, um232h
CURRENT: upload_protocol = dfu
Uploading .pio/build/wio_lite_risc-v/firmware.bin
dfu-util 0.9

Copyright 2005-2009 Weston Schmidt, Harald Welte and OpenMoko Inc.
Copyright 2010-2020 Tormod Volden and Stefan Schmidt
This program is Free Software and has ABSOLUTELY NO WARRANTY
Please report bugs to http://sourceforge.net/p/dfu-util/tickets/

Opening DFU capable USB device...
ID 28e9:0189
Run-time device DFU version 011a
Claiming USB DFU Interface...
Setting Alternate Setting #0 ...
Determining device status: state = dfuIDLE, status = 0
dfuIDLE, continuing
DFU mode device DFU version 011a
Device returned transfer size 2048
DfuSe interface name: "Internal Flash  "
Found GD32VF103, which reports a bad page size and count for its internal memory.
Fixed layout based on part number: page size 1024, count 128.
Downloading to address = 0x08000000, size = 9608

Erase   	[                         ]   0%            0 bytes
Erase   	[=====                    ]  21%         2048 bytes
Erase   	[==========               ]  42%         4096 bytes
Erase   	[===============          ]  63%         6144 bytes
Erase   	[=====================    ]  85%         8192 bytes
Download	[                         ]   0%            0 bytes
Download	[=====                    ]  21%         2048 bytes
Download	[==========               ]  42%         4096 bytes
Download	[===============          ]  63%         6144 bytes
Download	[=====================    ]  85%         8192 bytes
Download	[=========================] 100%         9608 bytes
Download done.
File downloaded successfully
dfu-util: Error during download get_status
*** [upload] Error 74
===== [FAILED] Took 2.43 seconds =====
#[FAILED]になっているが「File downloaded successfully」になっているので、書き込みが正常に終わっている。
```

これ以降、別のプログラムを動かすときは
wio_sampleのディレクトリをまるごと
コピーして別のプロジェクトのディレクトリを作り
そこにプログラム(.cpp)を置く。  

例：  

```bash

cp wio_sample wio_proj01
cd wio_proj01
...
```

## serial_test

src/serial_test.cpp
```c++


#include <Arduino.h>
#include <stdio.h>

#define CYCLES 2           // 2 yields a heartbeat effect
#define SHORT_DELAY  100   // 1/10 second
#define LONG_DELAY  1000   // 1 second


static void wio_serial_init(void)
{
    // enable GPIO clock 
    rcu_periph_clock_enable(RCU_GPIOA);
    // enable USART clock 
    rcu_periph_clock_enable(RCU_USART0);  

    // USART configure 
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

/*
// retarget the C library printf function to USART0
// Not needed, see the _put_char functino in
//   ~.platformio/packages/framework-arduino-gd32v/cores/arduino/GD32VF103_Firmware_Library/RISCV/stubs/write.c
extern "C" int _put_char(int ch) // used by printf
{
    usart_data_transmit(USART0, (uint8_t) ch );
    while (usart_flag_get(USART0, USART_FLAG_TBE) == RESET){
    }
    //delay(100);  // make it obvious the function is being used
    return ch;
}
*/

// the setup function runs once when you press reset or power the board
void setup() {
    // initialize digital pin LED_BUILTIN as an output.
    pinMode(LED_BUILTIN, OUTPUT);
   
    wio_serial_init();
    
    printf("\nSeeed Wio Lite RISC-V WiFi\n");
    printf("==========================\n");
    printf("Platform: Arduino\n");
    printf("LED_BUILTIN: %d\n", LED_BUILTIN);
    printf("\n");   
}

// the loop function runs over and over again forever
void loop() {
    for (int i=0; i<CYCLES; i++) {
      digitalWrite(LED_BUILTIN, HIGH);  // turn the red LED on
      printf("ON ");                    // update terminal
      delay(SHORT_DELAY);               // wait for a short on period
      digitalWrite(LED_BUILTIN, LOW);   // turn the red LED off
      delay(SHORT_DELAY);               // wait for a short off period
    }    
    digitalWrite(LED_BUILTIN, LOW);     // turn the LED off
    printf("OFF\n");                    // update terminal
    delay(LONG_DELAY);                  // wait for a long off period
}
```
ボードのPA9(TX). PA10(RX)、GNDをUSBシリアルに接続する。
書き込み後に「picocom /dev/ttyACM0 -b115200」で通信ソフトを起動すると以下のような出力が表示される：
```bash

$ picocom /dev/ttyACM0 -b115200

Seeed Wio Lite RISC-V WiFi
==========================
Platform: Arduino
LED_BUILTIN: 8

ON ON OFF
ON ON OFF
ON ON OFF
ON ON OFF
ON ON OFF
ON ON OFF
ON ON OFF
ON ON OFF
ON ON OFF
ON ON OFF
```
ちなみに、XIAOをUSBシリアルとして使用する場合は以下のように接続する：
| wio-lite-rv | XIAO |
| ---: | :--- |
| PA10(RX) | D6(TX) |
| PA9(TX) | D7(RX) |
| GND | GND |

## serial0_echo
シリアルエコープログラム:   
src/serial0_echo.cpp
```c++


#include <Arduino.h>
#include <stdio.h>
#include <stdarg.h>

static void wio_serial_init(void)
{
    // enable GPIO clock for USART0
    rcu_periph_clock_enable(RCU_GPIOA);
    // enable USART clock 
    rcu_periph_clock_enable(RCU_USART0);  
    /* connect port to USARTx_Tx(PA9) */ 
    gpio_init(GPIOA, GPIO_MODE_AF_PP, GPIO_OSPEED_50MHZ, GPIO_PIN_9);
    /* connect port to USARTx_Rx(PA10) */
    gpio_init(GPIOA, GPIO_MODE_IN_FLOATING, GPIO_OSPEED_50MHZ, GPIO_PIN_10);

    // USART0 configure 
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

int _get0_char(void) {
    while ( usart_flag_get(USART0, USART_FLAG_RBNE)== RESET){
    }
   return (uint8_t)usart_data_receive(USART0);
}

int _put0_char(int ch)
{
    usart_data_transmit(USART0, (uint8_t) ch );
    while ( usart_flag_get(USART0, USART_FLAG_TBE)== RESET){
    }
    return ch;
}

void setup() {
    wio_serial_init();
}

void loop() {
    _put0_char(_get0_char());
}
```
ボードのPA9(TX). PA10(RX)、GNDをUSBシリアルに接続する。
書き込み後に「picocom /dev/ttyACM0 -b115200」で通信ソフトを起動して
キー入力すると入力した文字が表示される。

## serial_2ch
シリアルを2ch有効にするプログラム：   
src/serial_2ch.cpp
```c++


#include <Arduino.h>
#include <stdio.h>
#include <stdarg.h>

int n = 0;

static void wio_serial_init(void)
{
    // enable GPIO clock for USART0
    rcu_periph_clock_enable(RCU_GPIOA);
    // enable USART clock 
    rcu_periph_clock_enable(RCU_USART0);  
    /* connect port to USARTx_Tx(PA9) */ 
    gpio_init(GPIOA, GPIO_MODE_AF_PP, GPIO_OSPEED_50MHZ, GPIO_PIN_9);
    /* connect port to USARTx_Rx(PA10) */
    gpio_init(GPIOA, GPIO_MODE_IN_FLOATING, GPIO_OSPEED_50MHZ, GPIO_PIN_10);

    // enable GPIO clock for USART1
    rcu_periph_clock_enable(RCU_GPIOA);
    // enable USART clock 
    rcu_periph_clock_enable(RCU_USART1); 
    /* connect port to USARTx_Tx(PA2) */ 
    gpio_init(GPIOA, GPIO_MODE_AF_PP, GPIO_OSPEED_50MHZ, GPIO_PIN_2);
    /* connect port to USARTx_Rx(PA3) */
    gpio_init(GPIOA, GPIO_MODE_IN_FLOATING, GPIO_OSPEED_50MHZ, GPIO_PIN_3);

    // USART0 configure 
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

    // USART1 configure 
    usart_deinit(USART1);
       
    usart_baudrate_set(USART1, 115200U);
    usart_word_length_set(USART1, USART_WL_8BIT);
    usart_stop_bit_set(USART1, USART_STB_1BIT);
    usart_parity_config(USART1, USART_PM_NONE);
    usart_hardware_flow_rts_config(USART1, USART_RTS_DISABLE);
    usart_hardware_flow_cts_config(USART1, USART_CTS_DISABLE);
    usart_receive_config(USART1, USART_RECEIVE_ENABLE);
    usart_transmit_config(USART1, USART_TRANSMIT_ENABLE);
    usart_enable(USART1);

}

int _get0_char(void) {
    while ( usart_flag_get(USART0, USART_FLAG_RBNE)== RESET){
    }
   return (uint8_t)usart_data_receive(USART0);
}

int _get1_char(void) {
    while ( usart_flag_get(USART1, USART_FLAG_RBNE)== RESET){
    }
   return (uint8_t)usart_data_receive(USART1);
}

int _put0_char(int ch)
{
    usart_data_transmit(USART0, (uint8_t) ch );
    while ( usart_flag_get(USART0, USART_FLAG_TBE)== RESET){
    }
 
    return ch;
}

int _put1_char(int ch)
{
    usart_data_transmit(USART1, (uint8_t) ch );
    while ( usart_flag_get(USART1, USART_FLAG_TBE)== RESET){
    }
 
    return ch;
}

void usart1_printf(const char *fmt, ...) {
    char buf[100];
    va_list args;
    va_start(args, fmt);
    vsprintf(buf, fmt, args);
    va_end(args);
 
    char *p = buf;
    while( *p != '\0' ) {
        _put1_char(*p);
        p++;
    }
}

void usart1_println(const char *fmt, ...) {
    char buf[100];
    va_list args;
    va_start(args, fmt);
    vsprintf(buf, fmt, args);
    va_end(args);
 
    char *p = buf;
    while( *p != '\0' ) {
        _put1_char(*p);
        p++;
    }
    _put1_char('\r');
    _put1_char('\n');
}


// the setup function runs once when you press reset or power the board
void setup() {
    wio_serial_init();
    
    printf("\nSerial#0\n");
    printf("#0==========================\n");

    usart1_printf("\r\nSerial#1\r\n");
    usart1_printf("#1==========================\r\n");
}

void loop() {
    printf("#0 line #%d\n", n);
    usart1_printf("#1 line #%d\r\n", n);
    n++;
}
```
・チャンネル＃０のボードのPA9(TX). PA10(RX)、GNDをUSBシリアルに接続する。   
・チャンネル＃1のボードのPA2(TX). PA3(RX)、GNDをUSBシリアルに接続する。  
(チャンネル＃1はオンボードの[ESP-WROOM-02]とシリアル接続されている)  

## AT_test
WiFiモジュール[ESP-WROOM-02]にシリアル#0経由でATコマンドを送るプログラム：   
src/AT_test.cpp
```c++


#include <Arduino.h>
#include <stdio.h>
#include <stdarg.h>

static void wio_serial_init(void)
{
    // enable GPIO clock for USART0
    rcu_periph_clock_enable(RCU_GPIOA);
    // enable USART clock 
    rcu_periph_clock_enable(RCU_USART0);  
    /* connect port to USARTx_Tx(PA9) */ 
    gpio_init(GPIOA, GPIO_MODE_AF_PP, GPIO_OSPEED_50MHZ, GPIO_PIN_9);
    /* connect port to USARTx_Rx(PA10) */
    gpio_init(GPIOA, GPIO_MODE_IN_FLOATING, GPIO_OSPEED_50MHZ, GPIO_PIN_10);

    // enable GPIO clock for USART1
    rcu_periph_clock_enable(RCU_GPIOA);
    // enable USART clock 
    rcu_periph_clock_enable(RCU_USART1); 
    /* connect port to USARTx_Tx(PA2) */ 
    gpio_init(GPIOA, GPIO_MODE_AF_PP, GPIO_OSPEED_50MHZ, GPIO_PIN_2);
    /* connect port to USARTx_Rx(PA3) */
    gpio_init(GPIOA, GPIO_MODE_IN_FLOATING, GPIO_OSPEED_50MHZ, GPIO_PIN_3);

    // USART0 configure 
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

    // USART1 configure 
    usart_deinit(USART1);
       
    usart_baudrate_set(USART1, 115200U);
    usart_word_length_set(USART1, USART_WL_8BIT);
    usart_stop_bit_set(USART1, USART_STB_1BIT);
    usart_parity_config(USART1, USART_PM_NONE);
    usart_hardware_flow_rts_config(USART1, USART_RTS_DISABLE);
    usart_hardware_flow_cts_config(USART1, USART_CTS_DISABLE);
    usart_receive_config(USART1, USART_RECEIVE_ENABLE);
    usart_transmit_config(USART1, USART_TRANSMIT_ENABLE);
    usart_enable(USART1);

}

int _get0_char(void) {
    while ( usart_flag_get(USART0, USART_FLAG_RBNE)== RESET){
    }
   return (uint8_t)usart_data_receive(USART0);
}

int _get1_char(void) {
    while ( usart_flag_get(USART1, USART_FLAG_RBNE)== RESET){
    }
   return (uint8_t)usart_data_receive(USART1);
}

int _put0_char(int ch)
{
    usart_data_transmit(USART0, (uint8_t) ch );
    while ( usart_flag_get(USART0, USART_FLAG_TBE)== RESET){
    }
 
    return ch;
}

int _put1_char(int ch)
{
    usart_data_transmit(USART1, (uint8_t) ch );
    while ( usart_flag_get(USART1, USART_FLAG_TBE)== RESET){
    }
 
    return ch;
}

void usart1_printf(const char *fmt, ...) {
    char buf[100];
    va_list args;
    va_start(args, fmt);
    vsprintf(buf, fmt, args);
    va_end(args);
 
    char *p = buf;
    while( *p != '\0' ) {
        _put1_char(*p);
        p++;
    }
}

void usart1_println(const char *fmt, ...) {
    char buf[100];
    va_list args;
    va_start(args, fmt);
    vsprintf(buf, fmt, args);
    va_end(args);
 
    char *p = buf;
    while( *p != '\0' ) {
        _put1_char(*p);
        p++;
    }
    _put1_char('\r');
    _put1_char('\n');
}


// the setup function runs once when you press reset or power the board
void setup() {
    wio_serial_init();
    
}

void loop() {
    if ( usart_flag_get(USART1, USART_FLAG_RBNE)== SET) _put0_char(_get1_char());
    if ( usart_flag_get(USART0, USART_FLAG_RBNE)== SET) _put1_char(_get0_char());
}
```
・チャンネル＃０のボードのPA9(TX). PA10(RX)、GNDをUSBシリアルに接続する。   
「picocom /dev/ttyACM0 -b115200」を起動して以下のコマンドを送る：  
(通信ソフトを起動した後、ボードを接続してプログラムを起動する)
```


AT+CWQAP
AT+CWMODE=1
AT+CWJAP="YOUR_SSID","YOUR_PASSWORD"
AT+CIFSR
AT+CIPMUX=1
AT+CIPSERVER=1,80
```
各行末には[Enter],[Control+J]を入力する。
YOUR_SSID,YOUR_PASSWORDの部分は、自分の環境に合わせた文字列を設定する。  
以下のような出力(レスポンス)が得られ、WiFiネットワークに接続できたのが分かる。
```

$ picocom /dev/ttyACM0 -b115200
...

2.0.0
max tx power=78,ret=0

AT+CWQAP
AT+CWMODE=1
AT+CWJAP="YOUR_SSID","YOUR_PASSWORD"
I (51523) wifi: state: 0 -> 2 (b0)
I (51528) wifi: state: 2 -> 3 (0)
I (51536) wifi: state: 3 -> 5 (10)
WIFI CONNECTED
WIFI GOT IP
AT+CIFSR
+CIFSR:STAIP,"192.168.0.28"
+CIFSR:STAMAC,"2c:f4:32:48:ed:0b"

OK
AT+CIPMUX=1

OK
AT+CIPSERVER=1,80

OK
# ここで、上で取得したIPと設定したportである80であるIP:port(192.168.0.28:80)にブラウザーでアクセスする
0,CONNECT

#　以下のようなレスポンスが返ってくる：
+IPD,0,464:GET / HTTP/1.1
Host: 192.168.0.28
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.80 Safari/537.36
Sec-Metadata: cause="forced", destination="document", site="cross-site"
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: ja-JP,ja;q=0.9,en-US;q=0.8,en;q=0.7

1,CONNECT
1,CLOSED
delete

OK
```


## 参考情報

[Wio Lite RISC-V(GD32VF103)](http://akizukidenshi.com/catalog/g/gM-14785/)   

[Wio Lite RISC V GD32VF103 with ESP8266](https://wiki.seeedstudio.com/Wio_Lite_RISC_V_GD32VF103_with_ESP8266/)  
[Blink with a Wio Lite RISC-V with ESP8266](https://www.sigmdel.ca/michel/ha/gd32v/wio_lite_01_en.html)  

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

以上

