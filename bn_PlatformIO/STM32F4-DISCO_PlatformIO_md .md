
2020/1/4

STM32F4-Discovery tool (VScode+PlatformIO)
# STM32F4-Discovery tool (VScode+PlatformIO)

## 概要
STM32F4-Discoveryボードの開発ツールとして、VScodeの拡張としてPlatformIOを導入して、そのなかで開発ツール(コンパイラ、リンカ、アップローダー)をインストールする。
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


新しいプロジェクトを作り、その際、今回の場合、Boardとして、   
「ST STM32F4DISCOVERY」を選択し、Frameworkとして   
「arduino」を選択して
Nameに任意の名前を入力して[Finish]をクリックする。   
(このNameはプロジェクト名かつフォルダ名になる)

その後、必要なツールのインストールが開始するが、完了後、platform.iniの内容に以下が含まれることを確認する。   
```

platform = ststm32
board = disco_f407vg
framework = arduino
```

## IDE設定
platformio.iniファイルを以下のように編集する：  
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

[env:disco_f407vg]
platform = ststm32
board = disco_f407vg
framework = arduino

upload_protocol = stlink

```


## ビルドテストのためのサンプルプログラム
LED点滅サンプル：   
```cpp

#include <Arduino.h>
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
```

## build/upload
(1)ボードとホストPCをUSBケーブルで接続する。   
(2)VScode画面の最下行にあるアイコンからBuidのアイコンをクリックしてビルド実行する。   
(3)VScode画面の最下行にあるアイコンからUploadのアイコンをクリックしてアップロード(ボードへのプログラム書込)を実行する。    
(4)書き込み終了後、ボードの[reset]ボタンを押して書き込んだプログラムを起動する。

build出力ログ例：
```

> Executing task in folder STM32F4DISCO_arduino_01: platformio run <

Processing disco_f407vg (platform: ststm32; board: disco_f407vg; framework: arduino)
-------------------------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/ststm32/disco_f407vg.html
PLATFORM: ST STM32 6.0.0 > ST STM32F4DISCOVERY
HARDWARE: STM32F407VGT6 168MHz, 128KB RAM, 1MB Flash
DEBUG: Current (stlink) On-board (stlink) External (blackmagic, jlink)
PACKAGES: toolchain-gccarmnoneeabi 1.70201.0 (7.2.1), framework-arduinoststm32 3.10700.191028 (1.7.0)
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 8 compatible libraries
Scanning dependencies...
No dependencies
Building in release mode
Checking size .pio/build/disco_f407vg/firmware.elf
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
DATA:    [          ]   0.7% (used 940 bytes from 131072 bytes)
PROGRAM: [          ]   1.3% (used 13380 bytes from 1048576 bytes)

===== [SUCCESS] Took 2.62 seconds =====

Terminal will be reused by tasks, press any key to close it.

```

upload出力ログ例：
```

> Executing task in folder STM32F4DISCO_arduino_01: platformio run --target upload <

Processing disco_f407vg (platform: ststm32; board: disco_f407vg; framework: arduino)
------------------------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/ststm32/disco_f407vg.html
PLATFORM: ST STM32 6.0.0 > ST STM32F4DISCOVERY
HARDWARE: STM32F407VGT6 168MHz, 128KB RAM, 1MB Flash
DEBUG: Current (stlink) On-board (stlink) External (blackmagic, jlink)
PACKAGES: tool-dfuutil 1.9.190708, toolchain-gccarmnoneeabi 1.70201.0 (7.2.1), tool-stm32duino 1.0.1, tool-openocd 2.1000.190707 (10.0), framework-arduinoststm32 3.10700.191028 (1.7.0)
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 8 compatible libraries
Scanning dependencies...
No dependencies
Building in release mode
Checking size .pio/build/disco_f407vg/firmware.elf
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
DATA:    [          ]   0.7% (used 940 bytes from 131072 bytes)
PROGRAM: [          ]   1.3% (used 13380 bytes from 1048576 bytes)
Configuring upload protocol...
AVAILABLE: blackmagic, jlink, mbed, stlink
CURRENT: upload_protocol = stlink
Uploading .pio/build/disco_f407vg/firmware.elf
xPack OpenOCD, 64-bit Open On-Chip Debugger 0.10.0+dev (2019-07-17-11:25)
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
debug_level: 1

srst_only separate srst_nogate srst_open_drain connect_deassert_srst

target halted due to debug-request, current mode: Thread 
xPSR: 0x01000000 pc: 0x080027f0 msp: 0x20020000
** Programming Started **
** Programming Finished **
** Verify Started **
** Verified OK **
** Resetting Target **
shutdown command invoked

===== [SUCCESS] Took 3.52 seconds =====

Terminal will be reused by tasks, press any key to close it.

```

## フォルダ構成
デフォルトの設定では、home/Documents/PlatformIO/Projectsのなかにプロジェクト名のフォルダが生成される。

## 参考URL  

[A new generation ecosystem for embedded development](https://platformio.org/)  
[ＳＴＭ３２Ｆ４ＤＩＳＣＯＶＥＲＹ](http://akizukidenshi.com/catalog/g/gM-05313/)   

## 備考
本ボードは「mbed enable」でないなど比較的古いボードであるが、PlatformIOで簡単にビルド＆アップロードできた。


以上
