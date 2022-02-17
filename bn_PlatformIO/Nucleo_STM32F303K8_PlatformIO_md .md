
2020/1/12

Nucleo STM32F303K8 tool (VScode+PlatformIO)
# Nucleo STM32F303K8 tool (VScode+PlatformIO)

## 概要
「Nucleo STM32F303K8」ボードの開発ツールとして、VScodeの拡張としてPlatformIOを導入して、そのなかで開発ツール(コンパイラ、リンカ、アップローダー)をインストールする。
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
「ST Nucleo F303K8」を選択し、Frameworkとして   
３つの選択肢(Arduino,Mbed,STM32Cube)から「mbed」を選択して
Nameに任意の名前を入力して[Finish]をクリックする。   
(このNameはプロジェクト名かつフォルダ名になる)

その後、必要なツールのインストールが開始するが、完了後、platform.iniの内容に以下が含まれることを確認する。   
```

[env:nucleo_f303k8]
platform = ststm32
board = nucleo_f303k8
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

[env:nucleo_f303k8]
platform = ststm32
board = nucleo_f303k8
framework = mbed

upload_port = /media/user/NODE_F303K8
upload_protocol = mbed

```

## ビルドテストのためのサンプルプログラム
サンプル(複数のプログラムを#defineで切り替える)：   
```cpp

#include <mbed.h>

//#define BLINK
#define SERIAL

#ifdef BLINK
// LED2(Red) for PWR, LED1 for COM(usb) on board
DigitalOut myled(LED3); // Green LED on board

int main() {
    while(1) {
        myled = 1;
        wait(0.4);
        myled = 0;
        wait(1);
    }
}
#endif
//=========================================

#ifdef SERIAL

Serial pc(SERIAL_TX, SERIAL_RX); // 9600 bauds
 
DigitalOut myled(LED3);
 
int main()
{
    int i = 1;
    pc.printf("Hello World !\n");
    while(1) {
        wait(1);
        pc.printf("This program runs since %d seconds.\n", i++);
        myled = !myled;
    }
}
#endif
//=========================================
```

## build/upload
(1)ボードとホストPCをUSBケーブルで接続する。   
(2)VScode画面の最下行にあるアイコンからBuildのアイコンをクリックしてビルド実行する。   
(3)VScode画面の最下行にあるアイコンからUploadのアイコンをクリックしてアップロード(ボードへのプログラム書込)を実行する。    
注意：SerialMoniter画面に切り替えている場合、その画面で、Ctrl+Cを押して、事前に閉じる必要がある。   
(4)書き込み終了後、自動的に書き込んだプログラムが起動する。
（起動しない場合や再起動したい場合、ボードの[reset]ボタンを押す)

build出力ログ例：
```

> Executing task in folder ST32F303K8_mbed_01: platformio run <

Processing nucleo_f303k8 (platform: ststm32; board: nucleo_f303k8; framework: mbed)
---------------------------------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/ststm32/nucleo_f303k8.html
PLATFORM: ST STM32 6.0.0 > ST Nucleo F303K8
HARDWARE: STM32F303K8T6 72MHz, 12KB RAM, 64KB Flash
DEBUG: Current (stlink) On-board (stlink) External (blackmagic, jlink)
PACKAGES: toolchain-gccarmnoneeabi 1.70201.0 (7.2.1), framework-mbed 5.51401.191023 (5.14.1)
Collecting mbed sources...
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 14 compatible libraries
Scanning dependencies...
No dependencies
Building in release mode
<省略>
Linking .pio/build/nucleo_f303k8/firmware.elf
Checking size .pio/build/nucleo_f303k8/firmware.elf
Building .pio/build/nucleo_f303k8/firmware.bin
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
DATA:    [=         ]  13.0% (used 1600 bytes from 12288 bytes)
PROGRAM: [====      ]  43.0% (used 28196 bytes from 65536 bytes)

===== [SUCCESS] Took 11.12 seconds =====

Terminal will be reused by tasks, press any key to close it.

```

upload出力ログ例：
```

> Executing task in folder ST32F303K8_mbed_01: platformio run --target upload <

Processing nucleo_f303k8 (platform: ststm32; board: nucleo_f303k8; framework: mbed)
-----------------------------------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/ststm32/nucleo_f303k8.html
PLATFORM: ST STM32 6.0.0 > ST Nucleo F303K8
HARDWARE: STM32F303K8T6 72MHz, 12KB RAM, 64KB Flash
DEBUG: Current (stlink) On-board (stlink) External (blackmagic, jlink)
PACKAGES: tool-dfuutil 1.9.190708, toolchain-gccarmnoneeabi 1.70201.0 (7.2.1), framework-mbed 5.51401.191023 (5.14.1), tool-stm32duino 1.0.1, tool-openocd 2.1000.190707 (10.0)
Collecting mbed sources...
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 14 compatible libraries
Scanning dependencies...
No dependencies
Building in release mode
Checking size .pio/build/nucleo_f303k8/firmware.elf
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
DATA:    [=         ]  13.0% (used 1600 bytes from 12288 bytes)
PROGRAM: [====      ]  43.0% (used 28196 bytes from 65536 bytes)
Configuring upload protocol...
AVAILABLE: blackmagic, jlink, mbed, stlink
CURRENT: upload_protocol = mbed
Looking for upload disk...
Use manually specified: /media/komatsu/NODE_F303K8
Uploading .pio/build/nucleo_f303k8/firmware.bin
Firmware has been successfully uploaded.
(Some boards may require manual hard reset)

===== [SUCCESS] Took 9.34 seconds ====

Terminal will be reused by tasks, press any key to close it.```
```
Serial Monitor出力例：
```
$ platformio device monitor --baud 9600
--- Miniterm on /dev/ttyACM0  9600,8,N,1 ---
--- Quit: Ctrl+C | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H ---
Hello World !
This program runs since 1 seconds.
This program runs since 2 seconds.
This program runs since 3 seconds.
This program runs since 4 seconds.
This program runs since 5 seconds.
This program runs since 6 seconds.

--- exit ---
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
[Boards » NUCLEO-F303K8](https://os.mbed.com/platforms/ST-Nucleo-F303K8/)   
[ＳＴＭ３２　Ｎｕｃｌｅｏ　Ｂｏａｒｄ　ＳＴＭ３２Ｆ３０３Ｋ８](http://akizukidenshi.com/catalog/g/gM-10172/)   


以上
