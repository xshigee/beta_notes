
2020/7/28   

PlatformIO cli Wio Terminal Image-Load
# PlatformIO cli Wio Terminal Image-Load

## 概要
以下を参考にして、Wio-Terminalで画像を表示する(Seeeduino-Wio-Terminal/Arduino版)  
(ホストPCとしてはubuntuを想定している)

https://wiki.seeedstudio.com/Wio-Terminal-LCD-Loading-Image/


## デモ用の画像データを作成する
以下を実行する：
```bash

mkdir bmpConvertor
cd bmpConvertor

curl -fSLO https://files.seeedstudio.com/wiki/Wio-Terminal/res/bmp_converter.py
#必要があれば以下をインストールする
pip3 install pillow

mkdir bmp
cd bmp
#デモ用に変換したい(PC用の24bitの)bmp(複数)ファイルを、このディレクトリに置く

cd ..
python3 bmp_converter
# 8bit-colorに変換したい場合、１を入力する
# 16bit-colorに変換したい場合、2を入力する

#変換後、bmp/rgb223 bmp/rgb565 のディレクトリに変換結果が置かれる
#必要なものをWio-TerminalのSDにコピーする
```

## プロジェクト image-load のディレクトリを作成する

```
mkdir image-load	
cd image-load
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

lib_ldf_mode = deep+
```

## 該当ライブラリのインストール
```bash

cd image-load
cd src
curl -fSLO https://files.seeedstudio.com/wiki/Wio-Terminal/res/RawImage.h
```

## 実行するスケッチ
src/main.ino
```c++

//#define C8  // 8bits Color
#define C16 // 16bits Color

#include "TFT_eSPI.h"
#include "Seeed_FS.h" //Including SD card library
#include "RawImage.h"  //Including image processing library
TFT_eSPI tft;
 
void setup() {
    //Initialise SD card
    if (!SD.begin(SDCARD_SS_PIN, SDCARD_SPI)) {
        while (1);
    }
    tft.begin();
    tft.setRotation(3);

#ifdef C8
    // 8bits(RGB332) image loading
    tft.setTextColor(TFT_BLACK,TFT_WHITE);
    tft.fillScreen(TFT_BLACK);
    drawImage<uint8_t>("rgb332/1186.bmp", 0, 0);
    drawImage<uint8_t>("rgb332/1187.bmp", 0, 0);
    drawImage<uint8_t>("rgb332/1188.bmp", 0, 0);
    drawImage<uint8_t>("rgb332/1189.bmp", 0, 0);
    drawImage<uint8_t>("rgb332/1190.bmp", 0, 0);
    drawImage<uint8_t>("rgb332/1191.bmp", 0, 0);
    drawImage<uint8_t>("rgb332/1192.bmp", 0, 0);
    drawImage<uint8_t>("rgb332/1193.bmp", 0, 0);
    drawImage<uint8_t>("rgb332/1194.bmp", 0, 0);
    drawImage<uint8_t>("rgb332/1194.bmp", 0, 0);
    drawImage<uint8_t>("rgb332/1195.bmp", 0, 0);
    drawImage<uint8_t>("rgb332/1196.bmp", 0, 0);
    drawImage<uint8_t>("rgb332/1197.bmp", 0, 0);
    drawImage<uint8_t>("rgb332/1198.bmp", 0, 0);
    drawImage<uint8_t>("rgb332/1199.bmp", 0, 0);
    drawImage<uint8_t>("rgb332/1200.bmp", 0, 0);
    drawImage<uint8_t>("rgb332/1201.bmp", 0, 0);
    drawImage<uint8_t>("rgb332/1202.bmp", 0, 0);
#endif

#ifdef C16
    // 16bits(RGB565) image loading
    tft.setTextColor(TFT_BLACK,TFT_WHITE);
    tft.fillScreen(TFT_BLACK);
    drawImage<uint16_t>("rgb565/1186.bmp", 0, 0);
    drawImage<uint16_t>("rgb565/1187.bmp", 0, 0);
    drawImage<uint16_t>("rgb565/1188.bmp", 0, 0);
    drawImage<uint16_t>("rgb565/1189.bmp", 0, 0);
    drawImage<uint16_t>("rgb565/1190.bmp", 0, 0);
    drawImage<uint16_t>("rgb565/1191.bmp", 0, 0);
    drawImage<uint16_t>("rgb565/1192.bmp", 0, 0);
    drawImage<uint16_t>("rgb565/1193.bmp", 0, 0);
    drawImage<uint16_t>("rgb565/1194.bmp", 0, 0);
    drawImage<uint16_t>("rgb565/1194.bmp", 0, 0);
    drawImage<uint16_t>("rgb565/1195.bmp", 0, 0);
    drawImage<uint16_t>("rgb565/1196.bmp", 0, 0);
    drawImage<uint16_t>("rgb565/1197.bmp", 0, 0);
    drawImage<uint16_t>("rgb565/1198.bmp", 0, 0);
    drawImage<uint16_t>("rgb565/1199.bmp", 0, 0);
    drawImage<uint16_t>("rgb565/1200.bmp", 0, 0);
    drawImage<uint16_t>("rgb565/1201.bmp", 0, 0);
    drawImage<uint16_t>("rgb565/1202.bmp", 0, 0);
#endif

}
 
void loop() {
}
```


## コンパイルと実行
```

# ボードをホストPCに接続する
# build&upload(flash)
pio run -t upload
# buildしないで書き込む場合は以下を実行する：
pio run -t nobuild -t upload -v
# -v は、詳細を表示するオプション
```
8ビットカラー表示と16ビットカラー表示の両方が動作するが、メモリの関係か同時には１つしか動作しないようだ。
(#defineで切り替える)
実行すると「動画もどき」で表示される。


## 参考情報

[PlatformWio Terminal/ LCDLoading Image](https://wiki.seeedstudio.com/Wio-Terminal-LCD-Loading-Image/)   
[Interactive-FaceをPlatformIOでコンパイルして実行する(Seeeduino-Wio-Terminal/Arduino版)](https://beta-notes.way-nifty.com/blog/2020/07/post-9f0475.html)   

[M5Stack の LCD に TFT_eSPI を使って日本語フォント "源真ゴシック" で表示する](https://qiita.com/ma2shita/items/fef1608fc9cf3a7bd46a#font)

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

