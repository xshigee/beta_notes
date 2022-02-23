
2020/4/29

PlatformIO cli Arduino-nano
# PlatformIO cli Arduino-nano

## 概要
開発ツールPlatformIOをcli(comand line interface)で使う(Arduino-nano版)。VisualCodeのプラグインとしてPlatformIOを使用することができるが、ここでは、cliとしての使い方について記する。
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

## テスト用プロジェクト sample を作成/実行する
```bash

#ターゲットボードのtarget名を検索する
# (ここではarduino-nanoを検索する)
pio boards | grep -i nano
#出力例：
nanoatmega168                ATMEGA168      16MHz        14KB      1KB     Arduino Nano ATmega168
nanoatmega328                ATMEGA328P     16MHz        30KB      2KB     Arduino Nano ATmega328
nanoatmega328new             ATMEGA328P     16MHz        30KB      2KB     Arduino Nano ATmega328 (New Bootloader)
nano_every     ATMEGA4809  16MHz        47.50KB  6KB    Arduino Nano Every
nano_33_iot                    SAMD21G18A   48MHz        256KB    32KB   NANO 33 IoT
nano32  
....

#target名として「nanoatmega328」が判明した

# プロジェクトsampleのディレクトリを作成する
mkdir sample	
cd sample
# 以下を実行して必要なファイルを作成する
pio init --board nanoatmega328

# テスト用のmain.cppを作成する
nano src/main.cpp
次項の内容に編集する：
```
```c++
#include <Arduino.h>

#define LED 13

void setup() {
  pinMode(LED, OUTPUT);
}

void loop() {
  digitalWrite(LED, HIGH);
  delay(100);
  digitalWrite(LED, LOW);
  delay(100);
}
```
続き：
```bash

# build
pio run

# ボードをホストPCに接続する
# build&upload(flash)
pio run -t upload
# buildしないで書き込む場合は以下を実行する：
pio run -t nobuild -t upload -v
# -v は、詳細を表示するオプション

# 以上で、基本的な操作としては完了となる

```

## 外部ライブラリのインストール方法
```bash

# 外部ライブラリを検索する
# (ここでは例として「neopixel」を検索する)
pio lib search neopixel
#出力例：
Found 35 libraries:

Adafruit NeoPixel
=================
#ID: 28
Arduino library for controlling single-wire-based LED pixels and strip.

Keywords: display
Compatible frameworks: Arduino
Compatible platforms: Atmel AVR, Atmel megaAVR, Atmel SAM, Espressif 32, Espressif 8266, GigaDevice GD32V, Infineon XMC, Intel ARC32, Kendryte K210, Microchip PIC32, Nordic nRF51, Nordic nRF52, ST STM32, ST STM8, Teensy, TI MSP430
Authors: Adafruit

<省略>

...


# "Adafruit NeoPixel"(#ID: 28)をインストールする
# 以下を実行する：
pio lib install 28
# または、以下を実行する：
pio lib install "Adafruit NeoPixel"

nano src/main.cpp
以下の内容に差し替える：
```

```c++

#include <Arduino.h>

#include <Adafruit_NeoPixel.h>
#ifdef __AVR__
  #include <avr/power.h>
#endif

#define PIN 6

// Parameter 1 = number of pixels in strip
// Parameter 2 = Arduino pin number (most are valid)
// Parameter 3 = pixel type flags, add together as needed:
//   NEO_KHZ800  800 KHz bitstream (most NeoPixel products w/WS2812 LEDs)
//   NEO_KHZ400  400 KHz (classic 'v1' (not v2) FLORA pixels, WS2811 drivers)
//   NEO_GRB     Pixels are wired for GRB bitstream (most NeoPixel products)
//   NEO_RGB     Pixels are wired for RGB bitstream (v1 FLORA pixels, not v2)
//   NEO_RGBW    Pixels are wired for RGBW bitstream (NeoPixel RGBW products)
Adafruit_NeoPixel strip = Adafruit_NeoPixel(60, PIN, NEO_GRB + NEO_KHZ800);

// IMPORTANT: To reduce NeoPixel burnout risk, add 1000 uF capacitor across
// pixel power leads, add 300 - 500 Ohm resistor on first pixel's data input
// and minimize distance between Arduino and first pixel.  Avoid connecting
// on a live circuit...if you must, connect GND first.


// Fill the dots one after the other with a color
void colorWipe(uint32_t c, uint8_t wait) {
  for(uint16_t i=0; i<strip.numPixels(); i++) {
    strip.setPixelColor(i, c);
    strip.show();
    delay(wait);
  }
}

// Input a value 0 to 255 to get a color value.
// The colours are a transition r - g - b - back to r.
uint32_t Wheel(byte WheelPos) {
  WheelPos = 255 - WheelPos;
  if(WheelPos < 85) {
    return strip.Color(255 - WheelPos * 3, 0, WheelPos * 3);
  }
  if(WheelPos < 170) {
    WheelPos -= 85;
    return strip.Color(0, WheelPos * 3, 255 - WheelPos * 3);
  }
  WheelPos -= 170;
  return strip.Color(WheelPos * 3, 255 - WheelPos * 3, 0);
}

void rainbow(uint8_t wait) {
  uint16_t i, j;

  for(j=0; j<256; j++) {
    for(i=0; i<strip.numPixels(); i++) {
      strip.setPixelColor(i, Wheel((i+j) & 255));
    }
    strip.show();
    delay(wait);
  }
}

// Slightly different, this makes the rainbow equally distributed throughout
void rainbowCycle(uint8_t wait) {
  uint16_t i, j;

  for(j=0; j<256*5; j++) { // 5 cycles of all colors on wheel
    for(i=0; i< strip.numPixels(); i++) {
      strip.setPixelColor(i, Wheel(((i * 256 / strip.numPixels()) + j) & 255));
    }
    strip.show();
    delay(wait);
  }
}

//Theatre-style crawling lights.
void theaterChase(uint32_t c, uint8_t wait) {
  for (int j=0; j<10; j++) {  //do 10 cycles of chasing
    for (int q=0; q < 3; q++) {
      for (uint16_t i=0; i < strip.numPixels(); i=i+3) {
        strip.setPixelColor(i+q, c);    //turn every third pixel on
      }
      strip.show();

      delay(wait);

      for (uint16_t i=0; i < strip.numPixels(); i=i+3) {
        strip.setPixelColor(i+q, 0);        //turn every third pixel off
      }
    }
  }
}

//Theatre-style crawling lights with rainbow effect
void theaterChaseRainbow(uint8_t wait) {
  for (int j=0; j < 256; j++) {     // cycle all 256 colors in the wheel
    for (int q=0; q < 3; q++) {
      for (uint16_t i=0; i < strip.numPixels(); i=i+3) {
        strip.setPixelColor(i+q, Wheel( (i+j) % 255));    //turn every third pixel on
      }
      strip.show();

      delay(wait);

      for (uint16_t i=0; i < strip.numPixels(); i=i+3) {
        strip.setPixelColor(i+q, 0);        //turn every third pixel off
      }
    }
  }
}


void setup() {
  // This is for Trinket 5V 16MHz, you can remove these three lines if you are not using a Trinket
  #if defined (__AVR_ATtiny85__)
    if (F_CPU == 16000000) clock_prescale_set(clock_div_1);
  #endif
  // End of trinket special code

  strip.begin();
  strip.setBrightness(50);
  strip.show(); // Initialize all pixels to 'off'
}

void loop() {
  // Some example procedures showing how to display to the pixels:
  colorWipe(strip.Color(255, 0, 0), 50); // Red
  colorWipe(strip.Color(0, 255, 0), 50); // Green
  colorWipe(strip.Color(0, 0, 255), 50); // Blue
//colorWipe(strip.Color(0, 0, 0, 255), 50); // White RGBW
  // Send a theater pixel chase in...
  theaterChase(strip.Color(127, 127, 127), 50); // White
  theaterChase(strip.Color(127, 0, 0), 50); // Red
  theaterChase(strip.Color(0, 0, 127), 50); // Blue

  rainbow(20);
  rainbowCycle(20);
  theaterChaseRainbow(50);
}

```
本プログラムは以下のリンクにあるサンプルの流用だが未定義エラーが出たので、関数の定義の順序を変更した。  

https://platformio.org/lib/show/28/Adafruit%20NeoPixel/examples    

ファイルタイプ.inoのものを.cppに変更し,「#include <Arduino.h>」を追加したが、Arduinoのコンパイラと挙動が異なるようだ。   
なお、nexopixelのDINをD6に接続する。(言うまでもないことだが、Vdd=5V,GNDも接続する)


続き：
```bash

# build/flash/exec
pio run -t upload

```
サンプルコードの入手のことを考慮すると、「pio lib xxxx」で検索するよりも
webブラウザーで「platformio lib xxxx」で検索し、見つけたライブラリのID番号で
インストールしたほうが良いかもしれない。

## ファイル位置
```bash

# buildの結果である firmware.* は以下にある：
$ ls .pio/build/nanoatmega328/firmware.*
.pio/build/nanoatmega328/firmware.elf  .pio/build/nanoatmega328/firmware.hex

# インストールした外部ライブラリは以下にある「
$ ls .pio/build/nanoatmega328/lib2c2/Adafruit\ NeoPixel_ID28/
Adafruit_NeoPixel.cpp.o  esp8266.c.o
```

## 参考情報

[Arduino Nano Pinout](https://components101.com/sites/default/files/component_pin/Arduino-Nano-Pinout.png)   

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

以上

