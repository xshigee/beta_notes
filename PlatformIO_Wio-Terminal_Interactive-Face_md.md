
2020/7/28   

PlatformIO cli Wio Terminal Interactive-Face
# PlatformIO cli Wio Terminal Interactive-Face

## 概要
以下のInteractive-FaceをPlatformIOでコンパイルして実行する(Seeeduino-Wio-Terminal/Arduino版)  
オリジナルのものはArduino-IDEでコンパイルして実行しているが、この記事ではPlatformioでコンパイルして実行してみる。
(ホストPCとしてはubuntuを想定している)

https://wiki.seeedstudio.com/Wio-Terminal-Interactive-Face/


## プロジェクト wiot-face のディレクトリを作成する

```
mkdir wiot-face	
cd wiot-face
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
## 該当ライブラリのインストールとスケッチのダウンロード
```bash

cd wiot-face
pio lib install https://github.com/Seeed-Studio/Seeed_Arduino_LIS3DHTR/archive/beta.zip

wget https://files.seeedstudio.com/wiki/Wio-Terminal/res/Emoji_face.zip
uzip Emoji_face.zip
#解凍したディレクトリのなかのfaceディレクトリごとWio-TerminalのSDにコピーする
cp Emoji_face/Emoji_face.ino  src/
cp Emoji_face/RawImage.h  src/

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
LCDに「顔」が表示されボタンを押すと表情がかわり
傾けると「目」の位置が変化する動作になればＯＫとなる。

## 実行したスケッチ(参考)

src/Emoji_face.ino 
```c++


#ifdef KENDRYTE_K210
#include <SPIClass.h>
#else
#include <SPI.h>
#endif

#include<string.h>
#include "Seeed_FS.h"
#include"LIS3DHTR.h"
#include"TFT_eSPI.h"
#include"RawImage.h"


LIS3DHTR<TwoWire>  lis;
TFT_eSPI tft;

#define debug(...)        //Serial.printf(__VA_ARGS__);
#define debug_begin(baud) //Serial.begin(baud); while (!Serial);
#define screen_height   240
#define screen_width    320


uint8_t raw[screen_height * screen_width];

void writeToBuffer(int x, int y, Raw8 * img) {
  // debug("%p\n", img);
  // debug("%d, %d\n", img->width, img->height);
  for (int j = y; j < y + img->height(); j++) {
    for (int i = x; i < x + img->width(); i++) {
      raw[j * screen_width + i] = img->get(i - x, j - y);
    }
  }
}


void clearBuffer() {
  memset(raw, 0, sizeof(raw));
}

void flushBuffer() {
  tft.pushImage(0, 0, tft.width(), tft.height(), raw);
}

bool flag_left = false;
void button_handler_left() {
  flag_left = true;
}

bool flag_middle = false;
void button_handler_middle() {
  flag_middle = true;
}

bool flag_right = false;
void button_handler_right() {
  flag_right = true;
}

#define FILLTER_N 20
int filter(int16_t val) {
  int32_t filter_sum = 0;
  for (uint8_t i = 0; i < FILLTER_N; i++) {
    filter_sum += val;
    delay(1);
  }
  return (int)(filter_sum / FILLTER_N);
}


const char* face[] = {"face/defaultface.bmp", "face/face0.bmp", "face/face1.bmp", "face/face2.bmp", "face/face3.bmp"};

const char* eye[] = {"face/cool.bmp", "face/happy.bmp", "face/heart.bmp", "face/robot.bmp", "face/sad.bmp",
                     "face/star.bmp", "face/cross.bmp", "face/kaws.bmp", "face/red.bmp"
                    };

void setup() {
  debug_begin(115200);
  pinMode(BUTTON_1, INPUT);
  pinMode(BUTTON_2, INPUT);
  pinMode(BUTTON_3, INPUT);
  attachInterrupt(digitalPinToInterrupt(BUTTON_1), button_handler_left, FALLING);
  attachInterrupt(digitalPinToInterrupt(BUTTON_2), button_handler_middle, FALLING);
  attachInterrupt(digitalPinToInterrupt(BUTTON_3), button_handler_right, FALLING);
  if (!SD.begin(SDCARD_SS_PIN, SDCARD_SPI, 16000000)) {
    while (1);
  }
  
  tft.begin();
  tft.setRotation(3);

  tft.fillScreen(TFT_BLACK);

  lis.begin(Wire1);
  lis.setOutputDataRate(LIS3DHTR_DATARATE_25HZ);
  lis.setFullScaleRange(LIS3DHTR_RANGE_2G);
}

int8_t eye_count = 0;

void loop() {
  float x_raw = lis.getAccelerationX();
  float y_raw = lis.getAccelerationY();

  int16_t x_value = 100 * x_raw;
  int16_t y_value = 100 * y_raw;

  delay(30);

  int16_t x_axis = map(filter(x_value), -15, 15, 73, 77); // mapped x-axis
  int16_t y_axis = map(filter(y_value), -15, 15, 53, 57); // mapped y-axis

  Raw8 * eyes = newImage<uint8_t>(eye[eye_count]);
  writeToBuffer(x_axis, y_axis, eyes);
  writeToBuffer(x_axis + 120, y_axis, eyes);
  delete[] eyes;

  Raw8 * mouth = newImage<uint8_t>("face/mouth.bmp");
  writeToBuffer(112, 175, mouth);
  delete[] mouth;
  flushBuffer();
  clearBuffer();

  if (flag_left) {
    flag_left = false;
    eye_count++;
    if (eye_count == 9) eye_count = 0;
  }
  if (flag_right) {
    flag_right = false;
    eye_count--;
    if (eye_count == -1) eye_count = 8;
  }

  if (flag_middle) {
    flag_middle = false;
    for (uint8_t cnt = 1; cnt < 5; cnt++) {
      drawImage<uint8_t>(face[cnt], 75, 55);
    }
  }

}
```
ボタン入力、イメージ表示などのコーディングの参考になる。

## RawImageライブラリ(参考)

src/RawImage.h 
```c++

  #pragma once
#include<stdint.h>
#include<SD/Seeed_SD.h>


/*
USAGE:

    // when use 8bit color.
    Raw8 * img8 = newImage<uint8_t>("path to sd card image.");

    // when use 16bit color.
    Raw16 * img16 = newImage<uint16_t>("path to sd card image.");

    // do some drawing.
    // img8->draw();

    // remember release it
    img8->release();
    img16->release();
 */

extern TFT_eSPI tft;

template<class type>
struct RawImage{
    type * ptr(){
        return (type *)(this + 1);
    }
    type get(int16_t x, int16_t y){
        return this->ptr()[y * width() + x];
    }
    void draw(size_t x = 0, size_t y = 0){
        tft.pushImage(x, y, width(), height(), ptr());
    }
    void release(){
        delete [] this;
    }
    int16_t width(){ return _width; }
    int16_t height(){ return _height; }
private:
    int16_t  _width;
    int16_t  _height;
};

typedef RawImage<uint8_t>  Raw8;
typedef RawImage<uint16_t> Raw16;

template<class type>
RawImage<type> * newImage(const char * path){
    typedef RawImage<type> raw;
    File f = SD.open(path, FILE_READ);
    if (!f){
        return nullptr;
    }
    int32_t size = f.size();
    raw   * mem = (raw *)new uint8_t[size];
    if (mem == nullptr){
        return nullptr;
    }
    f.read(mem, size);
    f.close();
    return mem;
}

template<class type>
void drawImage(const char * path, size_t x = 0, size_t y = 0){
    auto img = newImage<type>(path);
    img->draw(x, y);
    img->release();
}
```

## 参考情報

[PlatformWio Terminal/ LCDLoading Image](https://wiki.seeedstudio.com/Wio-Terminal-LCD-Loading-Image/)

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