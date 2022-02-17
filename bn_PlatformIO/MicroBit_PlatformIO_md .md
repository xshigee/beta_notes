
2020/1/1

micro:bit MBED tool (VScode+PlatformIO)
# micro:bit MBED tool (VScode+PlatformIO)

## 概要
ここではmicro:bitをMBEDボードのひとつとして扱い、MBED開発ツールとして、VScodeの拡張のPlatformIOを導入して、そのなかで開発ツール(コンパイラ、リンカ、アップローダー)をインストールする。
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
sudo usermod -a -G plugdev $USER
```

(3)PlatformIOのCLIのインストール   
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
「BBC micro:bit」を選択し、Frameworkとして「mbed」を選択して
Nameに任意の名前を入力して[Finish]をクリックする。   
(このNameはプロジェクト名かつフォルダ名になる)

その後、必要なツールのインストールが開始するが、完了後、platform.iniの内容に以下が含まれることを確認する。   
   
```

platform = nordicnrf51
board = bbcmicrobit
framework = mbed
```

## IDE設定
platformio.iniファイルを以下のように編集する(upload方法について設定を追加する):    
注意： upload_portは、microbitのビルドしたHEXファイルを書き込むフォルダ名になるので
自分の環境に合わせて変更すること。

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

[env:bbcmicrobit]
platform = nordicnrf51
board = bbcmicrobit
framework = mbed

upload_port = /media/user/CHIBIBIT
upload_protocol = mbed

;upload_protocol = cmsis-dap

```

## ビルドテストのためのサンプルプログラム
ボードのLEDを点滅させるサンプル：
```cpp


#include <mbed.h>

/*
LED Matrix
LED(X,Y)    X=0 X=1 X=2 X=3 X=4
Y=0 p13,p4  p14,p7  p13,p5  p14,p8  p13,p6
Y=1 p15,p7  p15,p8  p15,p9  p15,p10 p15,p11
Y=2 p14,p5  p13,p12 p14,p6  p15,p12 p14,p4
Y=3 p13,p11 p13,p10 p13,p9  p13,p8  p13,p7
Y=4 p15,p6  p14,p10 p15,p4  p14,p9  p15,p5
*/

// x=0 y=0
//DigitalOut row(P0_13);
//DigitalOut col(P0_4, 0);

// x=4 y=4
//DigitalOut row(P0_15);
//DigitalOut col(P0_5, 0);

// x=4 y=0
//DigitalOut row(P0_13);
//DigitalOut col(P0_6, 0);

// x=2 y=2
DigitalOut row(P0_14);
DigitalOut col(P0_6, 0);

int main() {
    while(1) {
        row = 1;
        wait(0.2);
        row = 0; 
        wait(0.2);
    }
}
```

## build/upload
(1)ボードとホストPCをUSBケーブルで接続する。   
(2)VScode画面の最下行にあるアイコンからBuidのアイコンをクリックしてビルド実行する。   
(3)VScode画面の最下行にあるアイコンからUploadのアイコンをクリックしてアップロード(ボードへのプログラム書込)を実行する。   
(4)書き込み終了するとHEX書き込み用のフォルダが再度現れるので、その後、ボードの[reset]ボタンを押して書き込んだプログラムを起動する。

build時の出力ログ例：
```

> Executing task in folder microbit_mbed_01: platformio run <

Processing bbcmicrobit (platform: nordicnrf51; board: bbcmicrobit; framework: mbed)
---------------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/nordicnrf51/bbcmicrobit.html
PLATFORM: Nordic nRF51 5.0.0 > BBC micro:bit
HARDWARE: NRF51822 16MHz, 16KB RAM, 256KB Flash
DEBUG: Current (cmsis-dap) On-board (cmsis-dap) External (jlink)
PACKAGES: toolchain-gccarmnoneeabi 1.70201.0 (7.2.1), framework-mbed 5.51105.190312 (5.11.5), tool-sreccat 1.164.0 (1.64)
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 4 compatible libraries
Scanning dependencies...
No dependencies
Building in release mode
Checking size .pio/build/bbcmicrobit/firmware.elf
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
DATA:    [=         ]   5.5% (used 904 bytes from 16384 bytes)
PROGRAM: [          ]   4.4% (used 11412 bytes from 262144 bytes)
===== [SUCCESS] Took 3.65 seconds =====

Terminal will be reused by tasks, press any key to close it.

```

upload時のログ例：
```

> Executing task in folder microbit_mbed_01: platformio run --target upload <

Processing bbcmicrobit (platform: nordicnrf51; board: bbcmicrobit; framework: mbed)
---------------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/nordicnrf51/bbcmicrobit.html
PLATFORM: Nordic nRF51 5.0.0 > BBC micro:bit
HARDWARE: NRF51822 16MHz, 16KB RAM, 256KB Flash
DEBUG: Current (cmsis-dap) On-board (cmsis-dap) External (jlink)
PACKAGES: tool-nrfjprog 1.90702.1 (9.7.2), toolchain-gccarmnoneeabi 1.70201.0 (7.2.1), framework-mbed 5.51105.190312 (5.11.5), tool-openocd 2.1000.190707 (10.0), tool-sreccat 1.164.0 (1.64)
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 4 compatible libraries
Scanning dependencies...
No dependencies
Building in release mode
Checking size .pio/build/bbcmicrobit/firmware.elf
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
DATA:    [=         ]   5.5% (used 904 bytes from 16384 bytes)
PROGRAM: [          ]   4.4% (used 11412 bytes from 262144 bytes)
Configuring upload protocol...
AVAILABLE: cmsis-dap, jlink, mbed
CURRENT: upload_protocol = mbed
Looking for upload disk...
Use manually specified: /media/komatsu/CHIBIBIT
Uploading .pio/build/bbcmicrobit/firmware.hex
Firmware has been successfully uploaded.
(Some boards may require manual hard reset)
===== [SUCCESS] Took 3.45 seconds =====

Terminal will be reused by tasks, press any key to close it.

```

## フォルダ構成
デフォルトの設定では、home/Documents/PlatformIO/Projectsのなかにプロジェクト名のフォルダが生成される。

## 参考URL  

[A new generation ecosystem for embedded development](https://platformio.org/)  
[arm MBED Boards » BBC micro:bit](https://os.mbed.com/platforms/Microbit/)   


以上
