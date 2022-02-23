
2020/7/14  
2020/7/2  

PlatformIO cli XIAO MCP9808
# PlatformIO cli XIAO MCP9808

## 概要
XIAOで以下のMCP9808(温度センサ)を使用する(Arduino版)。
開発ツールのインストールについては[開発ツールPlatformIOをcliで使う(Seeeduino-XIAO版)](https://beta-notes.way-nifty.com/blog/2020/06/post-6f19ad.html)を参照のこと。
(ホストPCとしてはubuntuを想定している)

[Adafruit MCP9808 Precision I2C Temperature Sensor Guide](https://learn.adafruit.com/adafruit-mcp9808-precision-i2c-temperature-sensor-guide)   


## 接続
以下の接続でボードを接続する：
| XIAO | MCP9808 |
| ----: | :--- |
| 3V3 | Vdd |
| GND | GND |
| SDA(D4) | SDA |
| SCL(D5) | SCL |
The default I2C address is 0x18

## 外部ライブラリのインストール方法
```

# 外部ライブラリを検索する
# (ここでは例として「MCP9808」を検索する)
「platformio lib MCP9808」のキーワードでwebで検索する。

「#820 Adafruit MCP9808 Library」が見つかるので、これをインストールする。名前と番号のどちらかでインストールできるが、名前のほうが同じ名前で異なるライブラリが登録されている可能性があるので番号でインストールする：

pio lib install 820
```

## デモ・プログラム

src/testmcp9808.ino
```c++

/**************************************************************************/
/*!
This is a demo for the Adafruit MCP9808 breakout
----> http://www.adafruit.com/products/1782
Adafruit invests time and resources providing this open source code,
please support Adafruit and open-source hardware by purchasing
products from Adafruit!
*/
/**************************************************************************/

#include <Wire.h>
#include "Adafruit_MCP9808.h"

// Create the MCP9808 temperature sensor object
Adafruit_MCP9808 tempsensor = Adafruit_MCP9808();

void setup() {
  //Serial.begin(9600);
  Serial.begin(115200);
  while (!Serial); //waits for serial terminal to be open, necessary in newer arduino boards.
  Serial.println("MCP9808 demo");
  
  // Make sure the sensor is found, you can also pass in a different i2c
  // address with tempsensor.begin(0x19) for example, also can be left in blank for default address use
  // Also there is a table with all addres possible for this sensor, you can connect multiple sensors
  // to the same i2c bus, just configure each sensor with a different address and define multiple objects for that
  //  A2 A1 A0 address
  //  0  0  0   0x18  this is the default address
  //  0  0  1   0x19
  //  0  1  0   0x1A
  //  0  1  1   0x1B
  //  1  0  0   0x1C
  //  1  0  1   0x1D
  //  1  1  0   0x1E
  //  1  1  1   0x1F
  if (!tempsensor.begin(0x18)) {
    Serial.println("Couldn't find MCP9808! Check your connections and verify the address is correct.");
    while (1);
  }
    
   Serial.println("Found MCP9808!");

  tempsensor.setResolution(3); // sets the resolution mode of reading, the modes are defined in the table bellow:
  // Mode Resolution SampleTime
  //  0    0.5°C       30 ms
  //  1    0.25°C      65 ms
  //  2    0.125°C     130 ms
  //  3    0.0625°C    250 ms
}

void loop() {
  Serial.println("wake up MCP9808.... "); // wake up MCP9808 - power consumption ~200 mikro Ampere
  tempsensor.wake();   // wake up, ready to read!

  // Read and print out the temperature, also shows the resolution mode used for reading.
  Serial.print("Resolution in mode: ");
  Serial.println (tempsensor.getResolution());
  float c = tempsensor.readTempC();
  float f = tempsensor.readTempF();
  Serial.print("Temp: "); 
  Serial.print(c, 4); Serial.print("*C\t and "); 
  Serial.print(f, 4); Serial.println("*F.");
  
  delay(2000);
  Serial.println("Shutdown MCP9808.... ");
  tempsensor.shutdown_wake(1); // shutdown MSP9808 - power consumption ~0.1 mikro Ampere, stops temperature sampling
  Serial.println("");
  delay(200);
}
```

## 出力例
```
picocom /dev/ttyACM0 -b115200

...
<省略>
...

MCP9808 demo
Found MCP9808!
wake up MCP9808.... 
Resolution in mode: 3
Temp: 31.8750*C	 and 89.3750*F.
Shutdown MCP9808.... 

wake up MCP9808.... 
Resolution in mode: 3
Temp: 31.8750*C	 and 89.3750*F.
Shutdown MCP9808.... 

wake up MCP9808.... 
Resolution in mode: 3
Temp: 31.9375*C	 and 89.4875*F.
Shutdown MCP9808.... 

wake up MCP9808.... 
Resolution in mode: 3
Temp: 31.8750*C	 and 89.3750*F.
Shutdown MCP9808.... 

wake up MCP9808.... 
Resolution in mode: 3
Temp: 31.9375*C	 and 89.4875*F.
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

seeedstudio.com  
[Seeeduino XIAO - Arduino 互換ボード - SAMD21 Cortex M0+](https://jp.seeedstudio.com/Seeeduino-XIAO-Arduino-Microcontroller-SAMD21-Cortex-M0+-p-4426.html)   
[wiki: Seeeduino Xiaoをはじめよう](https://wiki.seeedstudio.com/jp/Seeeduino-XIAO/)  

以上
