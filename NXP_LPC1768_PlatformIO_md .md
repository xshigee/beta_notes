
2020/1/3

NXP LPC1768 tool (VScode+PlatformIO)
# NXP LPC1768 tool (VScode+PlatformIO)

## 概要
LPC1768ボードの開発ツールとして、VScodeの拡張としてPlatformIOを導入して、そのなかで開発ツール(コンパイラ、リンカ、アップローダー)をインストールする。
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
「NXP mbed LPC1768」を選択し、Frameworkとして   
「mbed」を選択して
Nameに任意の名前を入力して[Finish]をクリックする。   
(このNameはプロジェクト名かつフォルダ名になる)

その後、必要なツールのインストールが開始するが、完了後、platform.iniの内容に以下が含まれることを確認する。   
```

[env:lpc1768]
platform = nxplpc
board = lpc1768
framework = mbed
```

## IDE設定
platformio.iniファイルを以下のように編集する：  
upload_portは、自分の環境に合わせて変更すること。   
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

[env:lpc1768]
platform = nxplpc
board = lpc1768
framework = mbed

upload_port = /media/user/MBED
upload_protocol = mbed

;upload_protocol = cmsis-dap

```


## ビルドテストのためのサンプルプログラム
LED点滅サンプル：   
```cpp


#include <mbed.h>

//DigitalOut myled(LED1);
DigitalOut myled(LED2);
//DigitalOut myled(LED3);
//DigitalOut myled(LED4);


int main() {
    while(1) {
        myled = 1;
        wait(0.2);
        myled = 0;
        wait(0.2);
    }
}

```

## build/upload
(1)ボードとホストPCをUSBケーブルで接続する。   
(2)VScode画面の最下行にあるアイコンからBuidのアイコンをクリックしてビルド実行する。   
(3)VScode画面の最下行にあるアイコンからUploadのアイコンをクリックしてアップロード(ボードへのプログラム書込)を実行する。    
注意：SerialMoniter画面に切り替えている場合、その画面で、Ctrl+Cを押して、事前に閉じる必要がある。   
(4)書き込み終了後、ボードの[reset]ボタンを押して書き込んだプログラムを起動する。

build出力ログ例：
```


> Executing task in folder LPC1768_mbed_01: platformio run <

Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/nxplpc/lpc1768.html
PLATFORM: NXP LPC 5.0.0 > NXP mbed LPC1768
HARDWARE: LPC1768 96MHz, 64KB RAM, 512KB Flash
DEBUG: Current (cmsis-dap) On-board (cmsis-dap) External (blackmagic, jlink)
PACKAGES: toolchain-gccarmnoneeabi 1.70201.0 (7.2.1), framework-mbed 5.51401.191023 (5.14.1)
Collecting mbed sources...
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 11 compatible libraries
Scanning dependencies...
No dependencies
Building in release mode
Checking size .pio/build/lpc1768/firmware.elf
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
DATA:    [=         ]   5.3% (used 3441 bytes from 65536 bytes)
PROGRAM: [=         ]   6.5% (used 34148 bytes from 524288 bytes)
===== [SUCCESS] Took 8.68 seconds =====

Terminal will be reused by tasks, press any key to close it.
```

upload出力ログ例：
```

> Executing task in folder LPC1768_mbed_01: platformio run --target upload <

Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/nxplpc/lpc1768.html
PLATFORM: NXP LPC 5.0.0 > NXP mbed LPC1768
HARDWARE: LPC1768 96MHz, 64KB RAM, 512KB Flash
DEBUG: Current (cmsis-dap) On-board (cmsis-dap) External (blackmagic, jlink)
PACKAGES: toolchain-gccarmnoneeabi 1.70201.0 (7.2.1), framework-mbed 5.51401.191023 (5.14.1)
Collecting mbed sources...
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 11 compatible libraries
Scanning dependencies...
No dependencies
Building in release mode
Checking size .pio/build/lpc1768/firmware.elf
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
DATA:    [=         ]   5.3% (used 3441 bytes from 65536 bytes)
PROGRAM: [=         ]   6.5% (used 34148 bytes from 524288 bytes)
Configuring upload protocol...
AVAILABLE: blackmagic, cmsis-dap, jlink, mbed
CURRENT: upload_protocol = mbed
Looking for upload disk...
Use manually specified: /media/komatsu/MBED
Uploading .pio/build/lpc1768/firmware.bin
Firmware has been successfully uploaded.
(Some boards may require manual hard reset)

===== [SUCCESS] Took 7.96 seconds =====

Terminal will be reused by tasks, press any key to close it.

```

## MBEDストレージに書き込めない(uploadできない)場合
実際に体験したが、該当のストレージがRead-Onlyになりuploadできなくなったことがある。この場合、該当のMBEDストレージを初期化する。   

[参考]mbedのFAQからの抜粋：    
・mbedに書き込みができなくなった  
・ファイルを消しても空き容量が小さくなってしまっている   
・mbed内に消せないファイルがあって困っている   

mbedをフォーマットしてしまうことで解決できます

稀にmbedのストレージエリアに書き込みができない/ファイルを消せない問題が起こることがあります．また不可視ファイルが溜まっていって，空き容量が少なくなってしまうこともあります.   
このような場合は，mbedをフォーマットしてしまう方法があります．    
フォーマットしても「MBED.HTM」は電源の再投入(USBの抜き差しなど)で再び現れます


## フォルダ構成
デフォルトの設定では、home/Documents/PlatformIO/Projectsのなかにプロジェクト名のフォルダが生成される。

## 参考URL  

[A new generation ecosystem for embedded development](https://platformio.org/)  
[ｍｂｅｄ　ＮＸＰ　ＬＰＣ１７６８評価キット](http://akizukidenshi.com/catalog/g/gM-03596/)   

## 備考
usbmountはアンインストールしたほうが良いようだ。本件の書き込みにおいて、アクセス権限が強化されたりして不便なことが多い気がする。    
uploadの選択肢としてcmsis-dapがあるが、ストレージにファームウェアが書き込むmbedのほうがハードウェア依存性が少ないようで安定して書き込める気がする。


以上
