
2021/1/11  
初版  

pio microbit_screen lib
# pio microbit_screen lib

## 概要
micro:bitで以下のmicrobit_screenライブラリを利用する(micro:bit-v2対応,linux版)  

・[https://github.com/ht-deko/microbit_Screen](https://github.com/ht-deko/microbit_Screen)  

開発ツールとしてはplatformioを使用するが、そのインストールについては以下を参照のこと：  

・[micro:bit Arduino/MBED開発ツール(v2)(micro:bit-v2対応,linux版)](https://beta-notes.way-nifty.com/blog/2021/01/post-e6d91a.html)  

## 該当ライブラリーのインストール方法
```bash

cd demo_sketch
cd src

wget https://raw.githubusercontent.com/ht-deko/microbit_Screen/master/microbit_Screen.cpp
wget https://raw.githubusercontent.com/ht-deko/microbit_Screen/master/microbit_Screen.h

```

オリジナルは、microbit-v1対応なので、
microbit_Screen.hを以下のように修正して
microbit-v1/v2両方対応にする。
src/microbit_Screen.h
```c++
<修正無しなので省略>

#ifndef microbit_Screen_h
#define microbit_Screen_h

#include <Arduino.h>

#ifdef MICROBIT
const uint8_t max_cols PROGMEM = 9;
const uint8_t max_rows PROGMEM = 3;
const uint8_t colCount PROGMEM = 5;
const uint8_t rowCount PROGMEM = 5;
const uint8_t cols[max_cols] PROGMEM = {3, 4, 10, 23, 24, 25, 9, 7, 6};
const uint8_t rows[max_rows] PROGMEM = {26, 27, 28};
#endif
#ifdef MICROBIT_V2
const uint8_t max_cols PROGMEM = 5;
const uint8_t max_rows PROGMEM = 5;
const uint8_t colCount PROGMEM = 5;
const uint8_t rowCount PROGMEM = 5;
const int cols[5] = {4, 7, 3, 6, 10};
const int rows[5] = {21, 22, 23, 24, 25};
#endif

const uint32_t darknessValue = 493;
const uint32_t brightnessValue = 20;

typedef struct TPoint {
  uint8_t x;
  uint8_t y;
} LED_POINT;

#ifdef MICROBIT
const LED_POINT LED_POS[rowCount][colCount] PROGMEM =
{
  {{0, 0}, {3, 1}, {1, 0}, {4, 1}, {2, 0}},
  {{3, 2}, {4, 2}, {5, 2}, {6, 2}, {7, 2}},
  {{1, 1}, {8, 0}, {2, 1}, {8, 2}, {0, 1}},
  {{7, 0}, {6, 0}, {5, 0}, {4, 0}, {3, 0}},
  {{2, 2}, {6, 1}, {0, 2}, {5, 1}, {1, 2}}
};
#endif
#ifdef MICROBIT_V2
const LED_POINT LED_POS[rowCount][colCount] PROGMEM =
{
  {{0, 0}, {1, 0}, {2, 0}, {3, 0}, {4, 0}},
  {{0, 1}, {1, 1}, {2, 1}, {3, 1}, {4, 1}},
  {{0, 2}, {1, 2}, {2, 2}, {3, 2}, {4, 2}},
  {{0, 3}, {1, 3}, {2, 3}, {3, 3}, {4, 3}},
  {{0, 4}, {1, 4}, {2, 4}, {3, 4}, {4, 4}}
};
#endif

const uint8_t LED_FONT[96][colCount] PROGMEM =
<修正無しなので省略>
```


## arduino用のplatformio.ini

microbit-v1用
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

[env:bbcmicrobit]
platform = nordicnrf51
board = bbcmicrobit
framework = arduino
build_flags = -DMICROBIT -DNRF51_S110
monitor_speed = 115200
lib_ldf_mode = deep+

upload_protocol = cmsis-dap

lib_deps = 
    https://github.com/adafruit/Adafruit_BusIO/archive/master.zip
    https://github.com/sparkfun/SparkFun_MAG3110_Breakout_Board_Arduino_Library/archive/master.zip
    https://cdn-learn.adafruit.com/assets/assets/000/046/217/original/MMA8653.zip
    https://github.com/stm32duino/LSM303AGR/archive/master.zip
    https://github.com/adafruit/Adafruit-GFX-Library/archive/master.zip
    #
    https://github.com/sandeepmistry/arduino-BLEPeripheral/archive/master.zip
    https://github.com/adafruit/Adafruit_Microbit/archive/master.zip

```

microbit-v2用
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

[env:bbcmicrobit_v2]
platform = nordicnrf52
board = bbcmicrobit_v2
framework = arduino
build_flags = -DMICROBIT_V2
monitor_speed = 115200
lib_ldf_mode = deep+

upload_protocol = cmsis-dap

lib_deps = 
    https://github.com/adafruit/Adafruit_BusIO/archive/master.zip
    https://github.com/sparkfun/SparkFun_MAG3110_Breakout_Board_Arduino_Library/archive/master.zip
    https://cdn-learn.adafruit.com/assets/assets/000/046/217/original/MMA8653.zip
    https://github.com/stm32duino/LSM303AGR/archive/master.zip
    https://github.com/adafruit/Adafruit-GFX-Library/archive/master.zip
    #
    #https://github.com/sandeepmistry/arduino-BLEPeripheral/archive/master.zip
    #https://github.com/adafruit/Adafruit_Microbit/archive/master.zip

```

## デモ・スケッチ
以下はv1/v2の両方で動作する：

src/demo.ino
```C++

#include "microbit_Screen.h"

void setup() {
  SCREEN.begin();
}

void loop() {
  //SCREEN.showString("HELLO,WORLD.");  
  //SCREEN.showString("micro:bit!");

// patched pattern for MB2
  SCREEN.showAnimation((String)
    ". # . . . # # # . . # # # . ." + 
    "# # . . . . . # . . . . # . ." + 
    ". # . . . . # . . . # # # . ." + 
    ". # . . . # . . . . . . # . ." + 
    ". # . . . # # # . . # # # . ."
  );
//  
  SCREEN.showIcon(IconNames::Heart);
}
```
microbit-V2では該当ライブラリの修正変更にバグがあり右側から２番目の列のLEDが光らないので
数字の１，２，３を表示した後、欠けたハートマークが表示される。
(今のところ原因不明)  
当然のことながらV1では正常に動作する。

## 参考情報

[BBC micro:bit/Arduino化に特化した話](https://ht-deko.com/arduino/microbit.html)  

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

以上
