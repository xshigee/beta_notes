
2020/7/13   
2020/7/7   

PlatformIO cli Wio Terminal
# PlatformIO cli Wio Terminal

## 概要
開発ツールPlatformIOをcli(comand line interface)で使う(Seeeduino-Wio-Terminal/Arduino版)。
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

## テスト用プロジェクト sample を作成/実行する
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

#target名として「seeed_wio_terminal」が判明した

# プロジェクト sample のディレクトリを作成する
mkdir sample	
cd sample
# 以下を実行して必要なファイルを作成する
pio init --board seeed_wio_terminal

# platformをupdateする
pio platform update

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

[env:seeed_wio_terminal]
platform = atmelsam
board = seeed_wio_terminal
framework = arduino

upload_protocol = sam-ba
```
```bash
# テスト用のblink.inoを作成する
nano src/blink.ino
以下のように編集する：
```
```c++


// the setup function runs once when you press reset or power the board
void setup() {
  // initialize digital pin LED_BUILTIN as an output.
  pinMode(LED_BUILTIN, OUTPUT);
}

// the loop function runs over and over again forever
void loop() {
  digitalWrite(LED_BUILTIN, HIGH);   // turn the LED on (HIGH is the voltage level)
  delay(100);                       // wait for a second
  digitalWrite(LED_BUILTIN, LOW);    // turn the LED off by making the voltage LOW
  delay(100);                       // wait for a second
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
本体内の青いLEDが点滅すれば動作としてはＯＫとなる。
(LEDの光は本体の穴から漏れて見える)

これ以降、別のプログラムを動かすときは
sampleのディレクトリをまるごと
コピーして別のプロジェクトのディレクトリを作り
そこにプログラム(.ino)を置く。  

例：  

```bash

cp sample wio_proj01
cd wio_proj01
...
```

## bootloader-mode
通常(/dev/ttyACM0が見えているとき)は、ツール側で自動的にresetしているので気にする必要はないが、(既に書き込んであるプログラムによっては)/dev/ttyACM0が見えない状況になり、ツール側で自動的にresetできなくなる。(=書き込みが失敗する)   
その場合はRESETボタンを２度素早く押すことで
Wio-Terminalにfirmwareを書き込めるモードになる。これを「bootloader mode」といい、このモードでは、USBストレージとしてArduinoディレクトリが現れる。  

補足：「stty -F /dev/ttyACM0」を実行することでボードをリセットしているようだが詳細は不明。


## Seeed-Studioライブラリのインストール

```bash

#標準的なもの
pio lib install "Adafruit Zero DMA Library"
pio lib install https://github.com/Seeed-Studio/Seeed_Arduino_SFUD/archive/master.zip
pio lib install https://github.com/Seeed-Studio/Seeed_Arduino_USBDISP/archive/master.zip
pio lib install https://github.com/Seeed-Studio/ArduinoCore-samd/archive/master.zip
pio lib install https://github.com/Seeed-Studio/Seeed_Arduino_LCD/archive/master.zip
pio lib install https://github.com/Seeed-Studio/Seeed_Arduino_FS/archive/master.zip
pio lib install https://github.com/Seeed-Studio/Seeed_Arduino_Linechart/archive/master.zip

#ネットワーク関連
pio lib install https://github.com/Seeed-Studio/Seeed_Arduino_atWiFi/archive/master.zip
pio lib install https://github.com/Seeed-Studio/Seeed_Arduino_FreeRTOS/archive/master.zip
pio lib install https://github.com/Seeed-Studio/Seeed_Arduino_atUnified/archive/master.zip
pio lib install https://github.com/Seeed-Studio/esp-at-lib/archive/develop.zip
pio lib install https://github.com/Seeed-Studio/Seeed_Arduino_mbedtls/archive/master.zip
pio lib install https://github.com/Seeed-Studio/Seeed_Arduino_atWiFiClientSecure/archive/master.zip

pio lib install https://github.com/Seeed-Studio/Seeed_Arduino_atHTTPClient/archive/master.zip
pio lib install https://github.com/Seeed-Studio/Seeed_Arduino_atWebServer/archive/master.zip


```

## インストールしたライブラリを使用したデモ・スケッチ

src/TFT_Mandlebrot.ino
```c++


#include <sfud.h>  // forced
// Mandlebrot

// This will run quite slowly due to the large number of floating point calculations per pixel

#include <TFT_eSPI.h> // Hardware-specific library
#include <SPI.h>

TFT_eSPI tft = TFT_eSPI();       // Invoke custom library

#define TFT_GREY 0x7BEF

unsigned long runTime = 0;

float sx = 0, sy = 0;
uint16_t x0 = 0, x1 = 0, yy0 = 0, yy1 = 0;

void setup() {
    Serial.begin(250000);
    //randomSeed(analogRead(A0));
    Serial.println();
    // Setup the LCD
    tft.init();
    tft.setRotation(3);
}

void loop() {
    runTime = millis();

    tft.fillScreen(TFT_BLACK);
    tft.startWrite();
    for (int px = 1; px < 320; px++) {
        for (int py = 0; py < 240; py++) {
            float x0 = (map(px, 0, 320, -250000 / 2,
                            -242500 / 2)) / 100000.0; //scaled x coordinate of pixel (scaled to lie in the Mandelbrot X scale (-2.5, 1))
            float yy0 = (map(py, 0, 240, -75000 / 4,
                             -61000 / 4)) / 100000.0; //scaled y coordinate of pixel (scaled to lie in the Mandelbrot Y scale (-1, 1))
            float xx = 0.0;
            float yy = 0.0;
            int iteration = 0;
            int max_iteration = 128;
            while (((xx * xx + yy * yy) < 4)  && (iteration < max_iteration)) {
                float xtemp = xx * xx - yy * yy + x0;
                yy = 2 * xx * yy + yy0;
                xx = xtemp;
                iteration++;
            }
            int color = rainbow((3 * iteration + 64) % 128);
            yield(); tft.drawPixel(px, py, color);
        }
    }
    tft.endWrite();

    Serial.println(millis() - runTime);
    while (1) {
        yield();
    }
}

unsigned int rainbow(int value) {
    // Value is expected to be in range 0-127
    // The value is converted to a spectrum colour from 0 = blue through to red = blue

    byte red = 0; // Red is the top 5 bits of a 16 bit colour value
    byte green = 0;// Green is the middle 6 bits
    byte blue = 0; // Blue is the bottom 5 bits

    byte quadrant = value / 32;

    if (quadrant == 0) {
        blue = 31;
        green = 2 * (value % 32);
        red = 0;
    }
    if (quadrant == 1) {
        blue = 31 - (value % 32);
        green = 63;
        red = 0;
    }
    if (quadrant == 2) {
        blue = 0;
        green = 63;
        red = value % 32;
    }
    if (quadrant == 3) {
        blue = 0;
        green = 63 - 2 * (value % 32);
        red = 31;
    }
    return (red << 11) + (green << 5) + blue;
}
```
実行するとLCDにマンデルブロート図形を描画する。

## 補足
直接関係ないがArduino-IDEで本記事のライブラリをインストールする場合、以下のようにする：
```

#arduino-x.x.xx配下のlibrariesディレクトリに必要なライブラリにダウンロードする：

cd ~/<省略>/arduino-1.8.12/libraries 
wget https://github.com/Seeed-Studio/Seeed_Arduino_atWiFi/archive/master.zip
unzip master.zip
rm master.zip

#以下に上と同様に必要なライブラリをダウンロードする
...
...
```

## 参考情報

[Wio Terminalをはじめよう(ピン配置、データシート、回路図など)](https://wiki.seeedstudio.com/jp/Wio-Terminal-Getting-Started/)  

[Platform/Wio Terminal/Network/Overview](https://wiki.seeedstudio.com/Wio-Terminal-Network-Overview/)  
download url:   
https://files.seeedstudio.com/wiki/Wio-Terminal/res/ameba-image-Tool-v2.4.1.zip  
https://files.seeedstudio.com/wiki/Wio-Terminal/res/20200601-rtl8720d-images-v2.2.0.0.zip  

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

https://wiki.seeedstudio.com/Wio-Terminal-Wi-Fi/  

\# ESP8266ライブラリと互換性があるので以下が参考になる：   
https://arduino-esp8266.readthedocs.io/en/latest/esp8266wifi/readme.html  
https://arduino-esp8266.readthedocs.io/en/latest/esp8266wifi/udp-examples.html  

以上

