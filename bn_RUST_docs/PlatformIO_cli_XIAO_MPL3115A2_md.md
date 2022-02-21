
2020/7/2

PlatformIO cli XIAO MPL115A2
# PlatformIO cli XIAO MPL115A2

## 概要
XIAOで以下のMPL3115A2(気圧・高度・気温センサ)を使用する(Arduino版)。
開発ツールのインストールについては[開発ツールPlatformIOをcliで使う(Seeeduino-XIAO版)](https://beta-notes.way-nifty.com/blog/2020/06/post-6f19ad.html)を参照のこと。
(ホストPCとしてはubuntuを想定している)

[MPL115A2 - I2C Barometric Pressure/Temperature Sensor](https://www.adafruit.com/product/992)   


## 接続
以下の接続でボードを接続する：
| XIAO | MPL3115A2 |
| ----: | :--- |
| 3V3 | Vin |
| GND | GND |
| SDA(D4) | SDA |
| SCL(D5) | SCL |
The default I2C address is 0x60

## 外部ライブラリのインストール方法
```

# 外部ライブラリを検索する
# (ここでは例として「MPL3115A2」を検索する)
「platformio lib MPL3115A2」のキーワードでwebで検索する。

「#796 Adafruit MPL3115A2 Library」が見つかるので、これをインストールする。名前と番号のどちらかでインストールできるが名前のほうは同じ名前で異なるライブラリが登録されている可能性があるので番号でインストールする：

pio lib install 796

#実際にコンパイルがエラーになったので、
#以下のように直接ライブラリのソースをダウンロードする：

cd src
wget https://raw.githubusercontent.com/adafruit/Adafruit_MPL3115A2_Library/master/Adafruit_MPL3115A2.cpp
wget https://raw.githubusercontent.com/adafruit/Adafruit_MPL3115A2_Library/master/Adafruit_MPL3115A2.h
```

## デモ・プログラム
src/testmpl3115a2.ino
```c++

/**************************************************************************/
/*!
    @file     Adafruit_MPL3115A2.cpp
    @author   K.Townsend (Adafruit Industries)
    @license  BSD (see license.txt)

    Example for the MPL3115A2 barometric pressure sensor

    This is a library for the Adafruit MPL3115A2 breakout
    ----> https://www.adafruit.com/products/1893

    Adafruit invests time and resources providing this open source code,
    please support Adafruit and open-source hardware by purchasing
    products from Adafruit!

    @section  HISTORY

    v1.0 - First release
*/
/**************************************************************************/

#include <Wire.h>
#include <Adafruit_MPL3115A2.h>

// Power by connecting Vin to 3-5V, GND to GND
// Uses I2C - connect SCL to the SCL pin, SDA to SDA pin
// See the Wire tutorial for pinouts for each Arduino
// http://arduino.cc/en/reference/wire
Adafruit_MPL3115A2 baro = Adafruit_MPL3115A2();

void setup() {
  //Serial.begin(9600);
  Serial.begin(115200);
  Serial.println("Adafruit_MPL3115A2 test!");
}

void loop() {
  if (! baro.begin()) {
    Serial.println("Couldnt find sensor");
    return;
  }
  
  float pascals = baro.getPressure();
  // Our weather page presents pressure in Inches (Hg)
  // Use http://www.onlineconversion.com/pressure.htm for other units
  //Serial.print(pascals/3377); Serial.println(" Inches (Hg)");
  Serial.print(pascals/100); Serial.println(" hPa");

  float altm = baro.getAltitude();
  Serial.print(altm); Serial.println(" meters");

  float tempC = baro.getTemperature();
  Serial.print(tempC); Serial.println("*C");

  Serial.println("");

  delay(250);
}

```

## 出力例
```
picocom /dev/ttyACM0 -b115200

...
<省略>
...

1002.59 hPa
89.19 meters
31.00*C

1002.59 hPa
89.25 meters
31.00*C

1002.64 hPa
88.87 meters
31.00*C

1002.69 hPa
88.94 meters
31.00*C

1002.64 hPa
88.75 meters
31.00*C

1002.61 hPa
88.81 meters
31.00*C
...

```

## 参考情報

[Seeeduino XIAO](http://akizukidenshi.com/catalog/g/gM-15178/)   

[XIAO Schematic](https://files.seeedstudio.com/wiki/Seeeduino-XIAO/res/Seeeduino-XIAO-v1.0-SCH-191112.pdf)  
[XIAO Pinout](https://www.electronics-lab.com/wp-content/uploads/2020/01/Seeeduino-XIAO-pinout.jpg)  

[Arduino Nano Pinout](https://components101.com/sites/default/files/component_pin/Arduino-Nano-Pinout.png)   

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

[Seeeduino XIAO Get Started By Nanase](https://wiki.seeedstudio.com/Seeeduino-XIAO-by-Nanase/)  
[コインサイズ Arduino互換機 Seeeduino XIAO を使ってみた](https://qiita.com/nanase/items/0fed598975c49b1d707e#spi-microsd%E3%82%AB%E3%83%BC%E3%83%89)  

以上
