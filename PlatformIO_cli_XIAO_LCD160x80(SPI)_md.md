
2020/7/16  

PlatformIO cli XIAO LCD160x80(SPI)
# PlatformIO cli XIAO LCD160x80(SPI)

## 概要
XIAOで以下のLCD160x80(SPI)を制御する(Arduino版)。
開発ツールのインストールについては[開発ツールPlatformIOをcliで使う(Seeeduino-XIAO版)](https://beta-notes.way-nifty.com/blog/2020/06/post-6f19ad.html)を参照のこと。
(ホストPCとしてはubuntuを想定している)

[waveshare - 0.96inch LCD Module](https://www.waveshare.com/wiki/0.96inch_LCD_Module)  


## 接続
以下のようにLCDとXIAOボードを接続する：
| LCD | CS    | D/C   | DIN   | CLK   | RST | VCC | GND | BL |
| :--: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| XIAO | D2 | D3 | D10(MOSI)  | D8(CSK) | D1 | 3V3 | GND | D0 |

本LCDモジュールでは、BL(バックライト)はオンにしても、それほど効果がないようなので
接続しない選択肢もある。（その場合、スケッチを変更する変更する必要あり）

## 外部ライブラリのインストール方法
本モジュールのコントローラはST7735なので、それ用のライブラリをインストールする：  
(ライブラリ名の混乱を回避するためにID番号でインストールする)
```

#pio lib install "Adafruit ST7735 and ST7789 Lib
pio lib install 12

#pio lib install "Adafruit GFX Library"
pio lib install 13

#テスト用スケッチをダウンロードする
cd src
wget https://raw.githubusercontent.com/adafruit/Adafruit-ST7735-Library/master/examples/graphicstest_hallowing_m0/graphicstest_hallowing_m0.ino
```

## テスト・プログラム
ダウンロードしたスケッチを以下の修正点に対応して変更する：   
修正点:
```
(1)コンパイルエラーを回避するために以下を先頭に追加する：
#include <Wire.h>
(2)XIAOのLCDモジュールの配線に合わせてピンアサインを修正する
// for XIAO
#define TFT_CS        2 // Hallowing display control pins: chip select
#define TFT_RST       1 // Display reset
#define TFT_DC        3 // Display data/command select
#define TFT_BACKLIGHT  0 // Display backlight pin
(3)オリジナルの画面サイズ128x128を実際のLCDサイズの160x80に変更する
  tft.initR(INITR_MINI160x80);        // Initialize LCD 160x80 screen
  tft.setRotation(1);
```

src/graphicstest_hallowing_m0.ino
```c++


#include <Wire.h> // forced
/**************************************************************************
  This is a library for several Adafruit displays based on ST77* drivers.

  Works with the HalloWing M0 Express
    ----> http://www.adafruit.com/products/3900

  Check out the links above for our tutorials and wiring diagrams.

  Adafruit invests time and resources providing this open source code,
  please support Adafruit and open-source hardware by purchasing
  products from Adafruit!

  Written by Limor Fried/Ladyada for Adafruit Industries.
  MIT license, all text above must be included in any redistribution
 **************************************************************************/

#include <Adafruit_GFX.h>    // Core graphics library
#include <Adafruit_ST7735.h> // Hardware-specific library for ST7735
#include <SPI.h>

// for XIAO
#define TFT_CS        2 // Hallowing display control pins: chip select
#define TFT_RST       1 // Display reset
#define TFT_DC        3 // Display data/command select
#define TFT_BACKLIGHT  0 // Display backlight pin
/*****************
#define TFT_CS        39 // Hallowing display control pins: chip select
#define TFT_RST       37 // Display reset
#define TFT_DC        38 // Display data/command select
#define TFT_BACKLIGHT  7 // Display backlight pin
*****************/

// OPTION 1 (recommended) is to use the HARDWARE SPI pins, which are unique
// to each board and not reassignable.
Adafruit_ST7735 tft = Adafruit_ST7735(TFT_CS, TFT_DC, TFT_RST);

// OPTION 2 lets you interface the display using ANY TWO or THREE PINS,
// tradeoff being that performance is not as fast as hardware SPI above.
//#define TFT_MOSI 29  // Data out
//#define TFT_SCLK 30  // Clock out
//Adafruit_ST7735 tft = Adafruit_ST7735(TFT_CS, TFT_DC, TFT_MOSI, TFT_SCLK, TFT_RST);

float p = 3.1415926;

void setup(void) {
  Serial.begin(9600);
  Serial.print(F("Hello! ST77xx TFT Test"));

  //tft.initR(INITR_HALLOWING);        // Initialize HalloWing-oriented screen

  tft.initR(INITR_MINI160x80);        // Initialize LCD 160x80 screen
  tft.setRotation(1);
  //or
  //tft.setRotation(3);

  pinMode(TFT_BACKLIGHT, OUTPUT);
  digitalWrite(TFT_BACKLIGHT, HIGH); // Backlight on

  Serial.println(F("Initialized"));

  uint16_t time = millis();
  tft.fillScreen(ST77XX_BLACK);
  time = millis() - time;

  Serial.println(time, DEC);
  delay(500);

  // large block of text
  tft.fillScreen(ST77XX_BLACK);
  testdrawtext("Lorem ipsum dolor sit amet, consectetur adipiscing elit. Curabitur adipiscing ante sed nibh tincidunt feugiat. Maecenas enim massa, fringilla sed malesuada et, malesuada sit amet turpis. Sed porttitor neque ut ante pretium vitae malesuada nunc bibendum. Nullam aliquet ultrices massa eu hendrerit. Ut sed nisi lorem. In vestibulum purus a tortor imperdiet posuere. ", ST77XX_WHITE);
  delay(1000);

  // tft print function!
  tftPrintTest();
  delay(4000);

  // a single pixel
  tft.drawPixel(tft.width()/2, tft.height()/2, ST77XX_GREEN);
  delay(500);

  // line draw test
  testlines(ST77XX_YELLOW);
  delay(500);

  // optimized lines
  testfastlines(ST77XX_RED, ST77XX_BLUE);
  delay(500);

  testdrawrects(ST77XX_GREEN);
  delay(500);

  testfillrects(ST77XX_YELLOW, ST77XX_MAGENTA);
  delay(500);

  tft.fillScreen(ST77XX_BLACK);
  testfillcircles(10, ST77XX_BLUE);
  testdrawcircles(10, ST77XX_WHITE);
  delay(500);

  testroundrects();
  delay(500);

  testtriangles();
  delay(500);

  mediabuttons();
  delay(500);

  Serial.println("done");
  delay(1000);
}

void loop() {
  tft.invertDisplay(true);
  delay(500);
  tft.invertDisplay(false);
  delay(500);
}

void testlines(uint16_t color) {
  tft.fillScreen(ST77XX_BLACK);
  for (int16_t x=0; x < tft.width(); x+=6) {
    tft.drawLine(0, 0, x, tft.height()-1, color);
    delay(0);
  }
  for (int16_t y=0; y < tft.height(); y+=6) {
    tft.drawLine(0, 0, tft.width()-1, y, color);
    delay(0);
  }

  tft.fillScreen(ST77XX_BLACK);
  for (int16_t x=0; x < tft.width(); x+=6) {
    tft.drawLine(tft.width()-1, 0, x, tft.height()-1, color);
    delay(0);
  }
  for (int16_t y=0; y < tft.height(); y+=6) {
    tft.drawLine(tft.width()-1, 0, 0, y, color);
    delay(0);
  }

  tft.fillScreen(ST77XX_BLACK);
  for (int16_t x=0; x < tft.width(); x+=6) {
    tft.drawLine(0, tft.height()-1, x, 0, color);
    delay(0);
  }
  for (int16_t y=0; y < tft.height(); y+=6) {
    tft.drawLine(0, tft.height()-1, tft.width()-1, y, color);
    delay(0);
  }

  tft.fillScreen(ST77XX_BLACK);
  for (int16_t x=0; x < tft.width(); x+=6) {
    tft.drawLine(tft.width()-1, tft.height()-1, x, 0, color);
    delay(0);
  }
  for (int16_t y=0; y < tft.height(); y+=6) {
    tft.drawLine(tft.width()-1, tft.height()-1, 0, y, color);
    delay(0);
  }
}

void testdrawtext(char *text, uint16_t color) {
  tft.setCursor(0, 0);
  tft.setTextColor(color);
  tft.setTextWrap(true);
  tft.print(text);
}

void testfastlines(uint16_t color1, uint16_t color2) {
  tft.fillScreen(ST77XX_BLACK);
  for (int16_t y=0; y < tft.height(); y+=5) {
    tft.drawFastHLine(0, y, tft.width(), color1);
  }
  for (int16_t x=0; x < tft.width(); x+=5) {
    tft.drawFastVLine(x, 0, tft.height(), color2);
  }
}

void testdrawrects(uint16_t color) {
  tft.fillScreen(ST77XX_BLACK);
  for (int16_t x=0; x < tft.width(); x+=6) {
    tft.drawRect(tft.width()/2 -x/2, tft.height()/2 -x/2 , x, x, color);
  }
}

void testfillrects(uint16_t color1, uint16_t color2) {
  tft.fillScreen(ST77XX_BLACK);
  for (int16_t x=tft.width()-1; x > 6; x-=6) {
    tft.fillRect(tft.width()/2 -x/2, tft.height()/2 -x/2 , x, x, color1);
    tft.drawRect(tft.width()/2 -x/2, tft.height()/2 -x/2 , x, x, color2);
  }
}

void testfillcircles(uint8_t radius, uint16_t color) {
  for (int16_t x=radius; x < tft.width(); x+=radius*2) {
    for (int16_t y=radius; y < tft.height(); y+=radius*2) {
      tft.fillCircle(x, y, radius, color);
    }
  }
}

void testdrawcircles(uint8_t radius, uint16_t color) {
  for (int16_t x=0; x < tft.width()+radius; x+=radius*2) {
    for (int16_t y=0; y < tft.height()+radius; y+=radius*2) {
      tft.drawCircle(x, y, radius, color);
    }
  }
}

void testtriangles() {
  tft.fillScreen(ST77XX_BLACK);
  uint16_t color = 0xF800;
  int t;
  int w = tft.width()/2;
  int x = tft.height()-1;
  int y = 0;
  int z = tft.width();
  for(t = 0 ; t <= 15; t++) {
    tft.drawTriangle(w, y, y, x, z, x, color);
    x-=4;
    y+=4;
    z-=4;
    color+=100;
  }
}

void testroundrects() {
  tft.fillScreen(ST77XX_BLACK);
  uint16_t color = 100;
  int i;
  int t;
  for(t = 0 ; t <= 4; t+=1) {
    int x = 0;
    int y = 0;
    int w = tft.width()-2;
    int h = tft.height()-2;
    for(i = 0 ; i <= 16; i+=1) {
      tft.drawRoundRect(x, y, w, h, 5, color);
      x+=2;
      y+=3;
      w-=4;
      h-=6;
      color+=1100;
    }
    color+=100; 
  }
}

void tftPrintTest() {
  tft.setTextWrap(false);
  tft.fillScreen(ST77XX_BLACK);
  tft.setCursor(0, 30);
  tft.setTextColor(ST77XX_RED);
  tft.setTextSize(1);
  tft.println("Hello World!");
  tft.setTextColor(ST77XX_YELLOW);
  tft.setTextSize(2);
  tft.println("Hello World!");
  tft.setTextColor(ST77XX_GREEN);
  tft.setTextSize(3);
  tft.println("Hello World!");
  tft.setTextColor(ST77XX_BLUE);
  tft.setTextSize(4);
  tft.print(1234.567);
  delay(1500);
  tft.setCursor(0, 0);
  tft.fillScreen(ST77XX_BLACK);
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(0);
  tft.println("Hello World!");
  tft.setTextSize(1);
  tft.setTextColor(ST77XX_GREEN);
  tft.print(p, 6);
  tft.println(" Want pi?");
  tft.println(" ");
  tft.print(8675309, HEX); // print 8,675,309 out in HEX!
  tft.println(" Print HEX!");
  tft.println(" ");
  tft.setTextColor(ST77XX_WHITE);
  tft.println("Sketch has been");
  tft.println("running for: ");
  tft.setTextColor(ST77XX_MAGENTA);
  tft.print(millis() / 1000);
  tft.setTextColor(ST77XX_WHITE);
  tft.print(" seconds.");
}

void mediabuttons() {
  // play
  tft.fillScreen(ST77XX_BLACK);
  tft.fillRoundRect(25, 10, 78, 60, 8, ST77XX_WHITE);
  tft.fillTriangle(42, 20, 42, 60, 90, 40, ST77XX_RED);
  delay(500);
  // pause
  tft.fillRoundRect(25, 90, 78, 60, 8, ST77XX_WHITE);
  tft.fillRoundRect(39, 98, 20, 45, 5, ST77XX_GREEN);
  tft.fillRoundRect(69, 98, 20, 45, 5, ST77XX_GREEN);
  delay(500);
  // play color
  tft.fillTriangle(42, 20, 42, 60, 90, 40, ST77XX_BLUE);
  delay(50);
  // pause color
  tft.fillRoundRect(39, 98, 20, 45, 5, ST77XX_RED);
  tft.fillRoundRect(69, 98, 20, 45, 5, ST77XX_RED);
  // play color
  tft.fillTriangle(42, 20, 42, 60, 90, 40, ST77XX_GREEN);
}
```
書き込み後、実行するとLCDにテスト画面が表示される。


## 参考情報

https://www.waveshare.com/wiki/File:ST7735S_V1.1_20111121.pdf   
https://www.waveshare.com/w/upload/3/35/0.96inch_LCD_Module_user_manual_en.pdf  

[Seeeduino XIAO](http://akizukidenshi.com/catalog/g/gM-15178/)   

[XIAO Schematic](https://files.seeedstudio.com/wiki/Seeeduino-XIAO/res/Seeeduino-XIAO-v1.0-SCH-191112.pdf)  
[XIAO Pinout](https://www.electronics-lab.com/wp-content/uploads/2020/01/Seeeduino-XIAO-pinout.jpg)  

[Arduino Nano Pinout](https://components101.com/sites/default/files/component_pin/Arduino-Nano-Pinout.png)   

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

[Seeeduino XIAO Get Started By Nanase](https://wiki.seeedstudio.com/Seeeduino-XIAO-by-Nanase/)  
[コインサイズ Arduino互換機 Seeeduino XIAO を使ってみた](https://qiita.com/nanase/items/0fed598975c49b1d707e#spi-microsd%E3%82%AB%E3%83%BC%E3%83%89)  

seeedstudio.com  
[Seeeduino XIAO - Arduino 互換ボード - SAMD21 Cortex M0+](https://jp.seeedstudio.com/Seeeduino-XIAO-Arduino-Microcontroller-SAMD21-Cortex-M0+-p-4426.html)   
[wiki: Seeeduino Xiaoをはじめよう](https://wiki.seeedstudio.com/jp/Seeeduino-XIAO/)  

以上
