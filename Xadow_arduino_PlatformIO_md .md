
2020/1/4

Xadow-arduino tool (VScode+PlatformIO)
# Xadow-arduino tool (VScode+PlatformIO)

## 概要
Xadowボードの開発ツールとして、VScodeの拡張としてPlatformIOを導入して、そのなかで開発ツール(コンパイラ、リンカ、アップローダー)をインストールする。
ここでは、linux環境でのインストール方法について説明する。   
注意：ここで取り上げるボードは、Xadowであって、Xadow-M0ではないので注意のこと。　本件のXadowボードは、Arduino-Leonardoボードと互換性がある。

## 準備
ツールをインストール前に環境整備として以下を設定する：  
(1)Python 2.7またはPython 3.5+のインストール
```bash

sudo apt-get install python3
```
(2)udevのruleの設定
```bash

curl -fsSL https://raw.githubusercontent.com/platformio/platformio-core/master/scripts/99-platformio-udev.rules | sudo tee /etc/udev/rules.d/99-platformio-udev.rules
sudo service udev restart
sudo usermod -a -G dialout $USER
sudo usermod -a -G plugdev $USER/home/komatsu/Documents/PlatformIO/Projects/LPC1768_mbed_01/src/main.cpp
```

## 開発ツールのインストール
上の準備が終わったら、プロセッサとボードの種類は異なるが、インストール方法が分かりやすいので、以下を参考にインストールする。：      
https://www.media.lab.uec.ac.jp/?page_id=1414  
ESP32をVSCodeとPlatformIO IDEで動かす方法   


新しいプロジェクトを作り、その際、今回の場合、Xadowそのものは存在しないので互換性のあるBoardとして、   
「Arduino Leonardo」を選択し、Frameworkとして   
「arduino」を選択して
Nameに任意の名前を入力して[Finish]をクリックする。   
(このNameはプロジェクト名かつフォルダ名になる)

その後、必要なツールのインストールが開始するが、完了後、platform.iniの内容に以下が含まれることを確認する。   
```

platform = atmelavr
board = leonardo
framework = arduino
```

## IDE設定
platformio.iniファイルを以下のようになっていることを確認する：  
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

[env:leonardo]
platform = atmelavr
board = leonardo
framework = arduino
```


## ビルドテストのためのサンプルプログラム
サンプル(#defineで二つのスケッチが切り替えられる)：   
```cpp

#include <Arduino.h>

#define XADOW_BLINKY
//#define ASCII_TABLE

#ifdef XADOW_BLINKY
/*
  Blink

  Turns an LED on for one second, then off for one second, repeatedly.

  Most Arduinos have an on-board LED you can control. On the UNO, MEGA and ZERO
  it is attached to digital pin 13, on MKR1000 on pin 6. LED_BUILTIN is set to
  the correct LED pin independent of which board is used.
  If you want to know what pin the on-board LED is connected to on your Arduino
  model, check the Technical Specs of your board at:
  https://www.arduino.cc/en/Main/Products

  modified 8 May 2014
  by Scott Fitzgerald
  modified 2 Sep 2016
  by Arturo Guadalupi
  modified 8 Sep 2016
  by Colby Newman

  This example code is in the public domain.

  http://www.arduino.cc/en/Tutorial/Blink
*/

//#define LED_XADOW 17 // Red LED on Xadow board
#define LED_XADOW 11 // Green LED on Xadow board
#define LED_BUILTIN LED_XADOW

// the setup function runs once when you press reset or power the board
void setup() {

  // initialize digital pin LED_BUILTIN as an output.
  pinMode(LED_BUILTIN, OUTPUT);; 
}

// the loop function runs over and over again forever
void loop() {
  digitalWrite(LED_BUILTIN, HIGH);   // turn the LED on (HIGH is the voltage level)
  delay(1000);                       // wait for a second
  digitalWrite(LED_BUILTIN, LOW);    // turn the LED off by making the voltage LOW
  delay(1000);                       // wait for a second
}
#endif
//=========================================================================

#ifdef ASCII_TABLE
/*
  ASCII table

  Prints out byte values in all possible formats:
  - as raw binary values
  - as ASCII-encoded decimal, hex, octal, and binary values

  For more on ASCII, see http://www.asciitable.com and http://en.wikipedia.org/wiki/ASCII

  The circuit: No external hardware needed.

  created 2006
  by Nicholas Zambetti <http://www.zambetti.com>
  modified 9 Apr 2012
  by Tom Igoe

  This example code is in the public domain.

  http://www.arduino.cc/en/Tutorial/ASCIITable
*/

void setup() {
  //Initialize serial and wait for port to open:
  Serial.begin(9600);
  while (!Serial) {
    ; // wait for serial port to connect. Needed for native USB port only
  }

  // prints title with ending line break
  Serial.println("ASCII Table ~ Character Map");
}

// first visible ASCIIcharacter '!' is number 33:
int thisByte = 33;
// you can also write ASCII characters in single quotes.
// for example, '!' is the same as 33, so you could also use this:
// int thisByte = '!';

void loop() {
  // prints value unaltered, i.e. the raw binary version of the byte.
  // The Serial Monitor interprets all bytes as ASCII, so 33, the first number,
  // will show up as '!'
  Serial.write(thisByte);

  Serial.print(", dec: ");
  // prints value as string as an ASCII-encoded decimal (base 10).
  // Decimal is the default format for Serial.print() and Serial.println(),
  // so no modifier is needed:
  Serial.print(thisByte);
  // But you can declare the modifier for decimal if you want to.
  // this also works if you uncomment it:

  // Serial.print(thisByte, DEC);


  Serial.print(", hex: ");
  // prints value as string in hexadecimal (base 16):
  Serial.print(thisByte, HEX);

  Serial.print(", oct: ");
  // prints value as string in octal (base 8);
  Serial.print(thisByte, OCT);

  Serial.print(", bin: ");
  // prints value as string in binary (base 2) also prints ending line break:
  Serial.println(thisByte, BIN);

  // if printed last visible character '~' or 126, stop:
  if (thisByte == 126) {    // you could also use if (thisByte == '~') {
    // This loop loops forever and does nothing
    while (true) {
      continue;
    }
  }
  // go on to the next character
  thisByte++;
}
//================================================
#endif

```

## build/upload
(1)ボードとホストPCをUSBケーブルで接続する。   
(2)VScode画面の最下行にあるアイコンからBuidのアイコンをクリックしてビルド実行する。   
(3)VScode画面の最下行にあるアイコンからUploadのアイコンをクリックしてアップロード(ボードへのプログラム書込)を実行する。    
(4)書き込み終了後、書き込んだプログラムが起動する。   
ボードのシリアル出力を見たい場合、VScode画面の最下行にある[Serial Monitor]をクリックする。

build出力ログ例：
```
> Executing task in folder Leonardo_01: platformio run <

Processing leonardo (platform: atmelavr; board: leonardo; framework: arduino)
----------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/atmelavr/leonardo.html
PLATFORM: Atmel AVR 2.0.0 > Arduino Leonardo
HARDWARE: ATMEGA32U4 16MHz, 2.50KB RAM, 28KB Flash
PACKAGES: framework-arduino-avr 5.0.0, toolchain-atmelavr 1.50400.190710 (5.4.0)
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 5 compatible libraries
Scanning dependencies...
No dependencies
Building in release mode
Compiling .pio/build/leonardo/src/main.cpp.o
src/main.cpp:33:0: warning: "LED_BUILTIN" redefined
 #define LED_BUILTIN LED_XADOW
 ^
In file included from /home/komatsu/.platformio/packages/framework-arduino-avr/cores/arduino/Arduino.h:257:0,
                 from src/main.cpp:1:
/home/komatsu/.platformio/packages/framework-arduino-avr/variants/leonardo/pins_arduino.h:106:0: note: this is the location of the previous definition
 #define LED_BUILTIN 13
 ^
# warningが出ているが特に動作には問題ない
Linking .pio/build/leonardo/firmware.elf
Checking size .pio/build/leonardo/firmware.elf
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
DATA:    [=         ]   5.8% (used 149 bytes from 2560 bytes)
PROGRAM: [=         ]  14.4% (used 4130 bytes from 28672 bytes)

===== [SUCCESS] Took 0.85 seconds =====

Terminal will be reused by tasks, press any key to close it.

```

upload出力ログ例：
```
> Executing task in folder Leonardo_01: platformio run --target upload <

Processing leonardo (platform: atmelavr; board: leonardo; framework: arduino)
----------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/atmelavr/leonardo.html
PLATFORM: Atmel AVR 2.0.0 > Arduino Leonardo
HARDWARE: ATMEGA32U4 16MHz, 2.50KB RAM, 28KB Flash
PACKAGES: framework-arduino-avr 5.0.0, toolchain-atmelavr 1.50400.190710 (5.4.0), tool-avrdude 1.60300.190424 (6.3.0)
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 5 compatible libraries
Scanning dependencies...
No dependencies
Building in release mode
Checking size .pio/build/leonardo/firmware.elf
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
DATA:    [=         ]   5.8% (used 149 bytes from 2560 bytes)
PROGRAM: [=         ]  14.4% (used 4130 bytes from 28672 bytes)
Configuring upload protocol...
AVAILABLE: avr109
CURRENT: upload_protocol = avr109
Looking for upload port...
Auto-detected: /dev/ttyACM0
Forcing reset using 1200bps open/close on port /dev/ttyACM0
Waiting for the new upload port...
Uploading .pio/build/leonardo/firmware.hex

Connecting to programmer: .
Found programmer: Id = "CATERIN"; type = S
    Software Version = 1.0; No Hardware Version given.
Programmer supports auto addr increment.
Programmer supports buffered memory access with buffersize=128 bytes.

Programmer supports the following devices:
    Device code: 0x44

avrdude: AVR device initialized and ready to accept instructions

Reading | ################################################## | 100% 0.00s

avrdude: Device signature = 0x1e9587 (probably m32u4)
avrdude: reading input file ".pio/build/leonardo/firmware.hex"
avrdude: writing flash (4130 bytes):

Writing | ################################################## | 100% 0.32s

avrdude: 4130 bytes of flash written
avrdude: verifying flash memory against .pio/build/leonardo/firmware.hex:
avrdude: load data flash data from input file .pio/build/leonardo/firmware.hex:
avrdude: input file .pio/build/leonardo/firmware.hex contains 4130 bytes
avrdude: reading on-chip flash data:

Reading | ################################################## | 100% 0.04s

avrdude: verifying ...
avrdude: 4130 bytes of flash verified

avrdude: safemode: Fuses OK (E:CE, H:D8, L:FF)

avrdude done.  Thank you.

===== [SUCCESS] Took 3.03 seconds =====

Terminal will be reused by tasks, press any key to close it.
```

## フォルダ構成
デフォルトの設定では、home/Documents/PlatformIO/Projectsのなかにプロジェクト名のフォルダが生成される。

## 参考情報  

[A new generation ecosystem for embedded development](https://platformio.org/)  
[Xadow - Main Board](https://www.seeedstudio.com/Xadow-Main-Board-p-1524.html)  
[Platform/RePhone/Xadow-M0](http://wiki.seeedstudio.com/Xadow_M0/)   
→本件のボードは、こっちではない。   
[Tutorial  Xadow IO pin mapping](http://wiki.seeedstudio.com/Xadow_IO_pin_mapping/)   
[Arduino Leonardo](https://ht-deko.com/arduino/leonardo.html)   
[Arduino Software I2C user guide](http://wiki.seeedstudio.com/Arduino_Software_I2C_user_guide/)   

## 備考
本ボードは既にディスコンしているようなので、新たに入手する際は、Xadow_M0をお勧めするが、mbedボードになるので、この記事の情報は役に立たない。

以上
