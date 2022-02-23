

2021/1/9+   
・BLE関連ライブラリでビルドできなかったが
platformio.iniを変更してビルド可能にした。      
・SoftDevice書き込みツールの
インストール方法を追加した。

2021/1/5  
SoftDevice(bleファームウェア)の再書き込みについて記載した。

2021/1/2  
初版  

cli micro:bit Arduino/MBED tool
# cli micro:bit Arduino/MBED tool

## 概要
micro:bit Arduino/MBED開発ツール(v2)(micro:bit-v2対応,linux版)

この記事は「
[micro:bit MBED開発ツール(VScode+PlatformIO)](https://beta-notes.way-nifty.com/blog/2020/01/post-1527c9.html)
 」の続編にあたり、arduinoプラットフォームでの設定やmicro:bit-v2の対応方法についても記述している。   
(ホストはubuntuを想定している)

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

## microbit-v2対応
microbit-v2の対応がまだ未完了のようなので、以下を行なう必要がある。
```
cd ~/.platformio/boards
# boardsがなければmkdirで作成する

wget https://raw.githubusercontent.com/platformio/platform-nordicnrf52/master/boards/bbcmicrobit_v2.json
```

bbcmicrobit_v2.jsonの内容にミスがあるので
以下のように修正する：
```
  "name": "BBC micro:bit V2",
  "upload": {
    "maximum_ram_size": 65536,
    "maximum_size": 524288,
    "protocol": "cmsis-dap",
    "protocols": [
      "jlink",

上を以下のように修正する：

  "name": "BBC micro:bit V2",
  "upload": {
    "maximum_ram_size": 131072,
    "maximum_size": 524288,
    "protocol": "cmsis-dap",
    "protocols": [
      "jlink",

```
RAMサイズが間違っていた。(64KB→128KB)  

コンパイル時(pio run)で以下のようなメッセージになればOK.
```

--------------------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/nordicnrf52/bbcmicrobit_v2.html
PLATFORM: Nordic nRF52 (6.0.0) > BBC micro:bit V2
HARDWARE: NRF52833 64MHz, 128KB RAM, 512KB Flash
DEBUG: Current (cmsis-dap) On-board (cmsis-dap) External (blackmagic, jlink, stlink)
PACKAGES: 
 - framework-arduinonordicnrf5 1.700.201209 (7.0) 
 - tool-bossac-nordicnrf52 1.10901.201022 (1.9.1) 
```

ちなみにv1の場合は以下のようになる：
```
--------------------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/nordicnrf51/bbcmicrobit.html
PLATFORM: Nordic nRF51 (6.1.0) > BBC micro:bit
HARDWARE: NRF51822 16MHz, 16KB RAM, 256KB Flash
DEBUG: Current (cmsis-dap) On-board (cmsis-dap) External (jlink)
PACKAGES: 
 - framework-arduinonordicnrf5 1.700.201209 (7.0) 
 - tool-nrfjprog 1.90702.1 (9.7.2) 
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
build_flags = -DMICROBIT
monitor_speed = 115200
lib_ldf_mode = deep+

#upload_port = /media/USER/MICROBIT
#upload_protocol = mbed
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
BLE関連を含むライブラリはコンパイルエラーになるのでコメントアウトしている。
コンパイルエラーになる原因は不明（今後、調査予定）(2021/1/9時点で解決)

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

#upload_port = /media/USER/MICROBIT
#upload_protocol = mbed
upload_protocol = cmsis-dap

lib_deps = 
    https://github.com/adafruit/Adafruit_BusIO/archive/master.zip
    https://github.com/sparkfun/SparkFun_MAG3110_Breakout_Board_Arduino_Library/archive/master.zip
    https://cdn-learn.adafruit.com/assets/assets/000/046/217/original/MMA8653.zip
    https://github.com/stm32duino/LSM303AGR/archive/master.zip
    https://github.com/adafruit/
    Adafruit-GFX-Library/archive/master.zip
    #
    #https://github.com/sandeepmistry/arduino-BLEPeripheral/archive/master.zip
    #https://github.com/adafruit/Adafruit_Microbit/archive/master.zip

```
BLE関連を含むライブラリはコンパイルエラーになるのでコメントアウトしている。
コンパイルエラーになるのはmicrobit-v2のプロセッサ対応が未完のせいが考えられる。

## テスト用スケッチ
以下はv1/v2の両方で動作する：

src/ASCIItable.ino
```C++

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
  Serial.begin(115200);
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
  // The Serial Monitor 
  interprets all bytes as ASCII, so 33, the first number,
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
```
arduinoのサンプルそのもの(bpsのみ変更)

src/microbit_blink.ino
```c++

#ifdef MICROBIT
const int COL1 = 3;     // Column #1 control
const int LED = 26;     // 'row 1' led
#endif

#ifdef MICROBIT_V2
const int COL1 = 4;
const int LED = 21; 
#endif
 
void setup() {  
  Serial.begin(115200);
  
  Serial.println("microbit is ready!");
 
  // because the LEDs are multiplexed, we must ground the opposite side of the LED
  pinMode(COL1, OUTPUT);
  digitalWrite(COL1, LOW); 
   
  pinMode(LED, OUTPUT);   
}
 
void loop(){
  Serial.println("blink!");
  
  digitalWrite(LED, HIGH);
  delay(100);
  digitalWrite(LED, LOW);
  delay(100);
}
```
MatrixLEDの仕様が変更になったので#ifdefで切り替えている。

## テスト用スケッチ(ビルドエラーが起きるもの)
以下は、platformioでライブラリがBLEに関係がありエラーになった例である：

src/matrixdemo.ino
```c++

// This is a demo of the matrix driver code
// works with Adafruit GFX commands - its just
// a really small screen!
// https://learn.adafruit.com/adafruit-gfx-graphics-library

#include <Adafruit_Microbit.h>

Adafruit_Microbit_Matrix microbit;

const uint8_t
  smile_bmp[] =
  { B00000,
    B01010,
    B00000,
    B10001,
    B01110, };
    
void setup() {  
  Serial.begin(9600);
  
  Serial.println("microbit matrix demo is ready!");

  microbit.begin();
}
    
void loop(){
  // Fill screen
  microbit.fillScreen(LED_ON);
  delay(1000);

  // draw a heart
  microbit.show(microbit.HEART);
  delay(1000);

  // draw a no cross
  microbit.show(microbit.NO);
  delay(1000);

  // draw a yes check
  microbit.show(microbit.YES);
  delay(1000);

  // draw a custom made bitmap face
  microbit.show(smile_bmp);
  delay(1000);
  
  microbit.clear();
  // Draw a line 'by hand'
  microbit.drawPixel(0, 0, LED_ON);
  microbit.drawPixel(1, 1, LED_ON);
  microbit.drawPixel(2, 2, LED_ON);
  microbit.drawPixel(3, 3, LED_ON);
  microbit.drawPixel(4, 4, LED_ON);
  // draw the 'opposite' line with drawline (easier!)
  microbit.drawLine(0, 4, 4, 0, LED_ON);

  delay(1000);

  // erase screen, draw a square
  microbit.clear();
  microbit.drawRect(0,0, 5, 5, LED_ON); // top left corner @ (0,0), 5 by 5 pixels size

  delay(1000);

  // erase screen, draw a circle
  microbit.clear();
  microbit.drawCircle(2,2, 2, LED_ON); // center on 2, 2, radius 2

  delay(1000);

  // erase screen, draw a filled triangle
  microbit.clear();
  microbit.fillTriangle(0,4, 2,0, 4,4, LED_ON); 

  delay(1000);

  // scroll some text the 'easy' way
  microbit.print("HELLO WORLD");

  // count up!
  for (int i=0; i<10; i++) {
    microbit.print(i);
    delay(500);
  }

  microbit.print(3.1415, 4);  // pi time, 4 digits of precision!!

}
```
本家のArduino-IDEではコンパイルができて実行もできた。(だだしv1のみ)

動作しない場合、SoftDevice(BLEファームウェア)が消されている可能性があるので   
以下でダウンロードしたhexファイルをMICROBITフォルダにドラッグ＆ドロップ後、  
書き込みを行なう。(Softdevice:"S110"にすることを忘れずに！)  
```

wget https://cdn-learn.adafruit.com/assets/assets/000/046/777/original/microbit-adv.hex

```
上の方法は、簡易的な方法になる。正式な書き込みツールのインストール方法については参考情報のリンクを参照のこと。   

src/ble_uartdemo.ino
```

/*
 * Serial Port over BLE
 * Create UART service compatible with Nordic's *nRF Toolbox* and Adafruit's *Bluefruit LE* iOS/Android apps.
 *
 * Copyright (c) Sandeep Mistry. All rights reserved.
 * Licensed under the MIT license. See LICENSE file in the project root for full license information.
 * BLESerial class implements same protocols as Arduino's built-in Serial class and can be used as it's wireless
 * replacement. Data transfers are routed through a BLE service with TX and RX characteristics. To make the
 * service discoverable all UUIDs are NUS (Nordic UART Service) compatible.
 *
 * Please note that TX and RX characteristics use Notify and WriteWithoutResponse, so there's no guarantee
 * that the data will make it to the other end. However, under normal circumstances and reasonable signal
 * strengths everything works well.
 */

#include <Adafruit_Microbit.h>

Adafruit_Microbit microbit;

void setup() {
  Serial.begin(115200);

  Serial.println("Microbit ready!");
  
  // custom services and characteristics can be added as well
  microbit.BTLESerial.begin();
  microbit.BTLESerial.setLocalName("microbit");

  // Start LED matrix driver after radio (required)
  microbit.begin();
}

void loop() {
  microbit.BTLESerial.poll();

  forward();
  //loopback();
  spam();
}


// forward received from Serial to microbit.BTLESerial and vice versa
void forward() {
  if (microbit.BTLESerial && Serial) {
    int byte;
    if (microbit.BTLESerial.available()) {
      Serial.write(microbit.BTLESerial.read());
    }
    char buffer[10];
    memset(buffer, 0x0, 10);
    int idx = 0;
    
    while (Serial.available() && idx != 10) {
       buffer[idx] = Serial.read();
       idx++;
    }
    if (idx) {
      microbit.BTLESerial.write(buffer, idx);
    }
  }
  delay(1);
}

// echo all received data back
void loopback() {
  if (microbit.BTLESerial) {
    int byte;
    while ((byte = microbit.BTLESerial.read()) > 0) {
        microbit.BTLESerial.write(byte);
    }
  }
}

// periodically sent time stamps
void spam() {　
  if (microbit.BTLESerial) {
    microbit.BTLESerial.print(millis());
    microbit.BTLESerial.println(" tick-tacks!");
    delay(1000);
  }
}
```
本家のArduino-IDEではコンパイルができて実行もできた。(だだしv1のみ)  

動作しない場合、SoftDevide(BLEファームウェア)が消されている可能性があるので   
以下でダウンロードしたhexファイルをMICROBITフォルダにドラッグ＆ドロップ後、  
書き込みを行なう。(Softdevice:"S110"にすることを忘れずに！)  
```

wget https://cdn-learn.adafruit.com/assets/assets/000/046/777/original/microbit-adv.hex

```
上の方法は、簡易的な方法になる。正式な書き込みツールのインストール方法については参考情報のリンクを参照のこと。   

対向するフマフォ・アプリは以下になる：  
・[Bluefruit Connect/iPhone](https://apps.apple.com/us/app/adafruit-bluefruit-le-connect/id830125974)    
・[Bluefruit Connect/Android](https://play.google.com/store/apps/details?id=com.adafruit.bluefruit.le.connect&hl=ja&gl=US)  

スマフォ・アプリ使用時、デバイスのMACアドレスだけでデバイス名がないので   
「Uart Capable」と付記があるアドレスのデバイスを選択すること。


## Arduino-IDEを使用する際の設定

```
(1)IDEを起動して[ファイル/環境設定(Preferences)]に入り
以下のurlを[追加のボードマネージャーのURL]のテキストボックスに以下を入力する。

https://sandeepmistry.github.io/arduino-nRF5/package_nRF5_boards_index.json

(2)[ツール/ボード/ボードマネージャ(Tools>Board>Boards Manager)]で「nRF5」で検索して
"Nordic Semiconductor nRF5 Boards" by Sandeep Mistry
をインストールする

(3)platformio.iniに登録されているzipのurlにアクセスしてzipファイルをダウンロードする

(4)[スケッチ/ライブラリをインクルード/.ZIP形式でインストール]でダウンロードしたzipファイルを指定してインストールする

```
注意：  
ボードとして
\[BBC micro:bit\]\(v1\)を選択した場合、Softdeviceは"S110"に設定すること。  
v2の場合、不要(設定できない)


ショートカット：
```
上の(3)(4)は以下の方法でも良い：

cd ~/Arduino/libraries
wget https://xxxxx/archive/master.zip
unzip master.zip
rm master.zip
# 以上をファイルの数だけ繰り返す
```

## mbed対応
上で述べたplatformio.iniのplatformのところを以下のように変更するとmbed対応になる。
```

framework = arduino

以下のように変更する：

framework = mbed

```
ただし、v2は、まだ、mbed対応になっていなのでエラーになる。
また、当然のことながら、arduino用ライブラリでなくmbed用ライブラリを使うこと。

また、microbit-v1用開発ツールとして以下のものを使用する選択肢もある：    
[micro:bit Yotta開発ツール(linux版)](https://beta-notes.way-nifty.com/blog/2021/01/post-6eed4c.html)   

## platformioでBLE関連ライブラリがビルドできなかった問題の解決(2021/1/9)
microbit-v1用のplatformio.iniを以下のように変更する：  
(変更箇所のみ記載) 
```

<省略>
build_flags = -DMICROBIT -DNRF51_S110
<省略>
    #
    https://github.com/sandeepmistry/arduino-BLEPeripheral/archive/master.zip
    https://github.com/adafruit/Adafruit_Microbit/archive/master.zip
<省略>
```

具体的には、ビルドできなかった以下のスケッチがビルド＆実行可能になる。  
```

src/matrixdemo.ino
src/ble_uartdemo.ino
```

ただし、ble_uartdemo.ino は参照エラーになるので   
ソースの先頭に以下のプロトタイプ宣言を追加すること：
```

// prototype definitions
void forward();
void spam();
void loopback();
```

## 参考情報

SoftDevice書き込みツールのインストール方法  
https://github.com/sandeepmistry/arduino-nRF5#flashing-a-softdevice
```

Flashing a SoftDevice

cd <SKETCHBOOK>, where <SKETCHBOOK> is your Arduino Sketch folder:

OS X: ~/Documents/Arduino
Linux: ~/Arduino
Windows: ~/Documents/Arduino

Create the following directories: tools/nRF5FlashSoftDevice/tool/

Download nRF5FlashSoftDevice.jar to <SKETCHBOOK>/tools/nRF5FlashSoftDevice/tool/

wget https://github.com/sandeepmistry/arduino-nRF5/releases/download/tools/nRF5FlashSoftDevice.jar

Restart the Arduino IDE
Select your nRF board from the Tools -> Board menu
Select a SoftDevice from the Tools -> "SoftDevice: " menu
Select a Programmer (J-Link, ST-Link V2, or CMSIS-DAP) from the Tools -> "Programmer: " menu
Select Tools -> nRF5 Flash SoftDevice

Read license agreement
Click "Accept" to accept license and continue, or "Decline" to decline and abort

If accepted, SoftDevice binary will be flashed to the board
```

[micro:bit v2 で遊ぶ](https://qiita.com/sat0ken/items/13bd03378c28b98a794e)   


platformio関連：
```

https://docs.platformio.org/en/latest/boards/nordicnrf52/bbcmicrobit_v2.html

https://docs.platformio.org/en/latest/platforms/creating_board.html

Installation
1.Create boards directory in core_dir if it doesn’t exist.
2.Create myboard.json file in this boards directory.
3.Search available boards via pio boards command. You should see myboard board.
```

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

以上
