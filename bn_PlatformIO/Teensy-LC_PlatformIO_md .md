
2020/1/6

Teensy-LC tool (VScode+PlatformIO)
# Teensy-LC tool (VScode+PlatformIO)

## 概要
Teensy-LCボードの開発ツールとして、VScodeの拡張としてPlatformIOを導入して、そのなかで開発ツール(コンパイラ、リンカ、アップローダー)をインストールする。
ここでは、linux環境でのインストール方法について説明する。   


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

新しいプロジェクトを作り、Boardとして、   
「Teensy LC」を選択し、Frameworkとして   
「arduino」を選択して
Nameに任意の名前を入力して[Finish]をクリックする。   
(このNameはプロジェクト名かつフォルダ名になる)

その後、必要なツールのインストールが開始するが、完了後、platform.iniの内容に以下が含まれることを確認する。   
```

platform = teensy
board = teensylc
framework = arduino
```

## IDE設定
platformio.iniファイルを以下のように編集する(デフォルトにupload_protocolを追加する)：  
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

[env:teensylc]
platform = teensy
board = teensylc
framework = arduino

upload_protocol = teensy-cli
```

## ビルドテストのためのサンプルプログラム
サンプル(#defineで二つのスケッチが切り替えられる)：   
```cpp

#include <Arduino.h>

#define BLINKY
//#define ASCII_TABLE

#ifdef BLINKY
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

// the setup function runs once when you press reset or power the board
void setup() {
  // initialize digital pin LED_BUILTIN as an output.
  pinMode(LED_BUILTIN, OUTPUT);
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

> Executing task in folder TeensyLC_ardiono_01: platformio run <

Processing teensylc (platform: teensy; board: teensylc; framework: arduino)
----------------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/teensy/teensylc.html
PLATFORM: Teensy 4.6.0 > Teensy LC
HARDWARE: MKL26Z64 48MHz, 8KB RAM, 62KB Flash
DEBUG: Current (jlink) External (jlink)
PACKAGES: framework-arduinoteensy 1.148.0 (1.48), toolchain-gccarmnoneeabi 1.50401.190816 (5.4.1)
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 89 compatible libraries
Scanning dependencies...
No dependencies
Building in release mode
Linking .pio/build/teensylc/firmware.elf
Checking size .pio/build/teensylc/firmware.elf
Building .pio/build/teensylc/firmware.hex
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
DATA:    [===       ]  25.4% (used 2084 bytes from 8192 bytes)
PROGRAM: [=         ]  10.4% (used 6584 bytes from 63488 bytes)

===== [SUCCESS] Took 1.24 seconds =====

Terminal will be reused by tasks, press any key to close it.
```

upload出力ログ例：
```

> Executing task in folder TeensyLC_ardiono_01: platformio run --target upload <

Processing teensylc (platform: teensy; board: teensylc; framework: arduino)
----------------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/teensy/teensylc.html
PLATFORM: Teensy 4.6.0 > Teensy LC
HARDWARE: MKL26Z64 48MHz, 8KB RAM, 62KB Flash
DEBUG: Current (jlink) External (jlink)
PACKAGES: framework-arduinoteensy 1.148.0 (1.48), tool-teensy 1.148.191118 (1.48), toolchain-gccarmnoneeabi 1.50401.190816 (5.4.1)
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 89 compatible libraries
Scanning dependencies...
No dependencies
Building in release mode
Linking .pio/build/teensylc/firmware.elf
Checking size .pio/build/teensylc/firmware.elf
Building .pio/build/teensylc/firmware.hex
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
DATA:    [===       ]  25.4% (used 2084 bytes from 8192 bytes)
PROGRAM: [=         ]  10.4% (used 6584 bytes from 63488 bytes)
Configuring upload protocol...
AVAILABLE: jlink, teensy-cli, teensy-gui
CURRENT: upload_protocol = teensy-cli
Rebooting...
Uploading .pio/build/teensylc/firmware.hex
Teensy Loader, Command Line, Version 2.1
Read ".pio/build/teensylc/firmware.hex": 6588 bytes, 10.4% usage
Unable to soft reboot with USB error: error sending control message: Broken pipe
Waiting for Teensy device...
 (hint: press the reset button)
Found HalfKay Bootloader
Read ".pio/build/teensylc/firmware.hex": 6588 bytes, 10.4% usage
Programming.............
Booting

===== [SUCCESS] Took 2.30 seconds =====

Terminal will be reused by tasks, press any key to close it.
```

## フォルダ構成
デフォルトの設定では、home/Documents/PlatformIO/Projectsのなかにプロジェクト名のフォルダが生成される。

## 参考情報  

[A new generation ecosystem for embedded development](https://platformio.org/)    
 [Teensy LC - Low Cost](https://www.pjrc.com/teensy/teensyLC.html)   
[Teensy USB Development Board](https://www.pjrc.com/teensy/)   
[Switch Science/Teensy LC](https://www.switch-science.com/catalog/2333/)   

## 備考
本ボードはArduinoボードとほとんど同じ使い勝手で使用でき。CPUがCortex-M0に置き換わっていることを意識させない。
mbedの知識も必要ないのでArduinoに馴染んでいる人には最適解だと思われる。

以上
