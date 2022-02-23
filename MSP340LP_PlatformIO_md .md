
2019/12/31

MSP430 LaunchPad tool (VScode+PlatformIO)
# MSP430 LaunchPad tool (VScode+PlatformIO)

## 概要
MSP340LaunchPadボードの開発ツールとして、VScodeの拡張としてPlatformIOを導入して、そのなかで開発ツール(コンパイラ、リンカ、アップローダー)をインストールする。
ここでは、linux環境でのインストール方法について説明する。upload(ファームの書き込み)については通常の方法ではエラーになり解決できなかったので、パッチ的な方法で書き込む。

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
sudo usermod -a -G plugdev $USER
```
(3)udevのruleの設定(Tiチップ関連)
```bash

curl -fsSL https://s3.amazonaws.com/energiaUS/files/71-ti-permissions.rules | sudo tee /etc/udev/rules.d/71-ti-permissions.rules
curl -fsSL https://aur.archlinux.org/cgit/aur.git/plain/61-msp430uif.rules?h=ccstudio | sudo tee /etc/udev/rules.d/61-msp430uif.rules
sudo service udev restart
sudo usermod -a -G dialout $USER
sudo usermod -a -G plugdev $USER
```
(3)upload用aliasの登録
.bashrcに以下を追加する
```bash

# alias for mspdebug of PlatformIO
alias mspg='mspdebug rf2500 "prog .pio/build/lpmsp430g2553/firmware.elf"'
```
(4)PlatformIOのCLIのインストール   
通常不要だがシェルからPlatformIOを使用したい場合、以下をインストールする：
```bash
sudo pip install -U pip
sudo pip install platformio
```

## 開発ツールのインストール
上の準備が終わったら、プロセッサとボードの種類は異なるが、インストール方法が分かりやすいので、以下を参考にインストールする。：      
https://www.media.lab.uec.ac.jp/?page_id=1414  
ESP32をVSCodeとPlatformIO IDEで動かす方法   

新しいプロジェクトを作り、その際、今回の場合、Boardとして、    
「TI LaunchPad MSP-EXP430G2553LP」(MSP430 Value Line LaunchPad)    
または   
「TI LaunchPad MSP-EXP430FR4133LP」(MSP430FR4133 LaunchPad)    
を選択し、Frameworkとして「Arduino」を選択して[Finish]をクリックする。

その後、必要なツールのインストールが開始するが、完了後、platform.iniの内容に以下が含まれることを確認する。   
(1)ボードが「MSP430 Value Line LaunchPad」の場合   
```

platform = timsp430
board = lpmsp430g2553
framework = arduino
```

(2)ボードが「MSP430FR4133 LaunchPad」の場合
```   

platform = timsp430
board = lpmsp430fr4133
framework = arduino
```

## IDE設定
platformio.iniファイルを以下のように編集する(upload方法について設定を追加する):    
(1)「MSP430 Value Line LaunchPad」の場合
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

[env:lpmsp430g2553]
platform = timsp430
board = lpmsp430g2553
framework = arduino

upload_protocol = mspdebug

```

(2)「MSP430FR4133 LaunchPad」の場合    
upload_portは実行環境で異なるので自分の環境のものに合わせること
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

[env:lpmsp430fr4133]
platform = timsp430
board = lpmsp430fr4133
framework = arduino

;upload_protocol = dslite

upload_port = /dev/ttyACM0
upload_speed= 115200

```

## ビルドテストのためのサンプルプログラム
arduinoのスケッチに「#include <Arduino.h>」を追加しただけで、大抵は動作するようだ。
ボードのLEDを点滅させるサンプル：
```cpp

#include <Arduino.h>
/*
  Blink
  The basic Energia example.
  Turns on an LED on for one second, then off for one second, repeatedly.
  Change the LED define to blink other LEDs.
  
  Hardware Required:
  * LaunchPad with an LED
  
  This example code is in the public domain.
*/

// most launchpads have a red LED
#define LED RED_LED

//see pins_energia.h for more LED definitions
//#define LED GREEN_LED
  
// the setup routine runs once when you press reset:
void setup() {                
  // initialize the digital pin as an output.
  pinMode(LED, OUTPUT);     
}

// the loop routine runs over and over again forever:
void loop() {
  digitalWrite(LED, HIGH);   // turn the LED on (HIGH is the voltage level)
  delay(100);               // wait for a second
  digitalWrite(LED, LOW);    // turn the LED off by making the voltage LOW
  delay(1000);               // wait for a second
}

```
「MSP430FR4133 LaunchPad」ボードのLCDを動かすサンプル：
```cpp

#include <Arduino.h>
//
//  Launchpad LCD
//  Example for library for Lauchpad LCD
//
//
//  Author :  Stefan Schauer
//  Date   :  June 23, 2014
//  Version:  1.00
//  File   :  LCD_Launchpad_demo.ino
//
//

// Include application, user and local libraries
 #include "LCD_Launchpad.h"

// Variables
LCD_LAUNCHPAD myLCD;

// Add setup code
void setup() {
    myLCD.init();
    
    myLCD.displayText("Hello");
    
    delay(1000);
    myLCD.clear();

    myLCD.showSymbol(LCD_SEG_RADIO, 1);
    delay(400);
    myLCD.showSymbol(LCD_SEG_TX, 1);
    delay(400);
    myLCD.showSymbol(LCD_SEG_RX, 1);
    delay(400);
	
    myLCD.showSymbol(LCD_SEG_HEART, 1);
    delay(400);
    myLCD.showSymbol(LCD_SEG_MARK, 1);
    delay(400);
    myLCD.showSymbol(LCD_SEG_CLOCK, 1);
    delay(400);
    myLCD.showSymbol(LCD_SEG_R, 1);
    delay(400);

    myLCD.showSymbol(LCD_SEG_BAT_ENDS, 1);
    delay(400);
    myLCD.showSymbol(LCD_SEG_BAT_POL, 1);
    delay(1000);
    myLCD.showSymbol(LCD_SEG_BAT0, 1);
    delay(200);
    myLCD.showSymbol(LCD_SEG_BAT1, 1);
    delay(200);
    myLCD.showSymbol(LCD_SEG_BAT2, 1);
    delay(200);
    myLCD.showSymbol(LCD_SEG_BAT3, 1);
    delay(200);
    myLCD.showSymbol(LCD_SEG_BAT4, 1);
    delay(200);
    myLCD.showSymbol(LCD_SEG_BAT5, 1);
    delay(200);
    myLCD.showSymbol(LCD_SEG_BAT_POL, 1);
    delay(1000);
    myLCD.showSymbol(LCD_SEG_BAT0, 0);
    delay(200);
    myLCD.showSymbol(LCD_SEG_BAT1, 0);
    delay(200);
    myLCD.showSymbol(LCD_SEG_BAT2, 0);
    delay(200);
    myLCD.showSymbol(LCD_SEG_BAT3, 0);
    delay(200);
    myLCD.showSymbol(LCD_SEG_BAT4, 0);
    delay(200);
    myLCD.showSymbol(LCD_SEG_BAT5, 0);
    delay(200);

    myLCD.showSymbol(LCD_SEG_MINUS1, 1);
    delay(400);
    myLCD.showSymbol(LCD_SEG_DOT1, 1);
    delay(400);
    myLCD.showSymbol(LCD_SEG_COLON2, 1);
    delay(400);
    myLCD.showSymbol(LCD_SEG_DOT2, 1);
    delay(400);
    myLCD.showSymbol(LCD_SEG_DOT3, 1);
    delay(400);
    myLCD.showSymbol(LCD_SEG_COLON4, 1);
    delay(400);
    myLCD.showSymbol(LCD_SEG_DOT4, 1);
    delay(400);
    myLCD.showSymbol(LCD_SEG_DOT5, 1);
    delay(400);
    myLCD.showSymbol(LCD_SEG_DEG5, 1);
    delay(1000);
}

// Add loop code
void loop() {
	uint16_t count = 0;
	
    myLCD.clear();
    myLCD.displayText("ABCDE");
    delay(1000);

    myLCD.clear();
    count = 0;
    while (count <= 150){
      myLCD.println(count++);
      delay(50);
    }
    delay(1000);
    
    myLCD.clear();
    myLCD.showChar('1', 1);
    myLCD.showChar('3', 3);
    myLCD.showChar('5', 5);
    delay(1000);
    myLCD.clear();
    myLCD.print("543");
    delay(1000);
    myLCD.println("21");
    delay(1000);
    myLCD.print("XYZ\nxyz");
    delay(1000);
    myLCD.println("");
    myLCD.clear();
}

```
## build/upload
(1)ボードとホストPCをUSBケーブルで接続する。   
(2)VScode画面の最下行にあるアイコンからBuidのアイコンをクリックしてビルド実行する。   
(3-1)lpmsp430g2553ボードの場合   
uploadが正常に動作しないので、以下の方法でuploadする。  
ビルドが完了したらVScodeのterminal画面をクリックして改行を押しトップレベルに入り    
以下のコマンドを実行する：  
mspg

(3-2)lpmsp430fr4133ボードの場合  
このボードは通常の方法でupload可能なので以下の手続きを行なう：   
VScode画面の最下行にあるアイコンからUploadのアイコンをクリックしてアップロード(ボードへのプログラム書込)を実行する。

(4)書き込み終了後、必要があればボードの[reset]ボタンを押して書き込んだプログラムを起動する。

upload出力ログ例(lpmsp430g2553ボード)：
```
mspg
MSPDebug version 0.25 - debugging tool for MSP430 MCUs
Copyright (C) 2009-2017 Daniel Beer <dlbeer@gmail.com>
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
Chip info database from MSP430.dll v3.3.1.4 Copyright (C) 2013 TI, Inc.

Trying to open interface 1 on 008
rf2500: warning: can't detach kernel driver: No data available
Initializing FET...
FET protocol version is 30394216
Set Vcc: 3000 mV
Configured for Spy-Bi-Wire
fet: FET returned error code 4 (Could not find device or device not supported)
fet: command C_IDENT1 failed
Using Olimex identification procedure
Device ID: 0x2553
  Code start address: 0xc000
  Code size         : 16384 byte = 16 kb
  RAM  start address: 0x200
  RAM  end   address: 0x3ff
  RAM  size         : 512 byte = 0 kb
Device: MSP430G2xx3
Number of breakpoints: 2
fet: FET returned NAK
warning: device does not support power profiling
Chip ID data:
  ver_id:         5325
  ver_sub_id:     0000
  revision:       00
  fab:            60
  self:           0000
  config:         00
  fuses:          00
Device: MSP430G2xx3
Erasing...
Programming...
Writing 2206 bytes at c000 [section: .text]...
Writing  158 bytes at c89e [section: .rodata]...
Writing    2 bytes at c93c [section: .data]...
Writing   32 bytes at ffe0 [section: .vectors]...
Done, 2398 bytes total

```

upload出力ログ例(lpmsp430fr4133ボード)：
```

Configuring upload protocol...
AVAILABLE: dslite
CURRENT: upload_protocol = dslite
Uploading .pio/build/lpmsp430fr4133/firmware.elf
DSLite version 8.2.0.1400
Configuring Debugger (may take a few minutes on first launch)...
        Initializing Register Database...
        Initializing: MSP430
        Executing Startup Scripts: MSP430
Connecting...
Loading Program: .pio/build/lpmsp430fr4133/firmware.elf
        Preparing ... 
        .text: 0 of 3278 at 0xc400
        .text: 0 of 128 at 0xff80: 95%
        Finished: 95%
        Setting PC to entry point.: 95%
info: MSP430:  Flash/FRAM usage is 3434 bytes. RAM usage is 0 bytes.
Running...
Success
===== [SUCCESS] Took 7.61 seconds =====
#以下はモニター用serial-portの設定なので使用しない場合は無視して良い。
komatsu@T530:~/Documents/PlatformIO/Projects/MSP430FR4133_arduino_01$ platformio device monitor --baud 9600

--- Available ports:
---  1: /dev/ttyACM0         MSP Tools Driver - MSP Debug Interface
---  2: /dev/ttyACM1         MSP Tools Driver - MSP Application UART
--- Enter port index or full name:
```

## フォルダ構成
デフォルトの設定では、home/Documents/PlatformIO/Projectsのなかにプロジェクト名のフォルダが生成される。

## 参考URL  
[ＭＳＰ４３０　Ｖａｌｕｅ　Ｌｉｎｅ　ＬａｕｎｃｈＰａｄ　Ｄｅｖｅｌｏｐｍｅｎｔ　Ｔｏｏｌ](http://akizukidenshi.com/catalog/g/gM-09662/)   
[ＭＳＰ４３０ＦＲ４１３３　ＬａｕｎｃｈＰａｄ　Ｄｅｖｅｌｏｐｍｅｎｔ　Ｋｉｔ](http://akizukidenshi.com/catalog/g/gM-09661/)   
[A new generation ecosystem for embedded development](https://platformio.org/)  
[https://energia.nu/](https://energia.nu/)   

sample code:  
[LCD_LaunchPad_demo.ino](https://github.com/energia/Energia/blob/master/hardware/msp430/libraries/LCD_Launchpad/examples/LCD_Launchpad_demo/LCD_Launchpad_demo.ino)

## おまけ(mspdebugのビルド方法)
uploadのエラーを解決するための試行錯誤の一環としてmspdebugの最新版をビルドしたので、その方法を以下に記載する。ただし、今回の書き込み方法において最新版にする必要はなかった。
```bash
sudo apt-get install libusb-dev
wget https://github.com/dlbeer/mspdebug/archive/v0.25.tar.gz
tar xvfz v0.25.tar.gz
cd mspdebug-0.25
make
sudo rm /usr/bin/mspdebug
sudo cp mspdebug /usr/bin/mspdebug

```
## 今回のパッチ的upload方法について
通常のuploadでエラーが出たので、ネット検索したら、ここで記載の方法でuploadが成功した旨の情報があったので、そのまま採用した。これで上手く行く理屈の正確なところは、わからないが、存在していないデバイス(fr2500)を指定することで、mspdebugがフォールバックを起こして、現在接続しているデバイスを検出することで、正常に動作するようになると推定している。同じmspdebugでenergiaのツールでは、問題なく動作していることから、mspdebugから見た動作環境に違いがあると思っている。なので、正常動作させるには、その動作環境の差分を解消する必要がある。

以上
