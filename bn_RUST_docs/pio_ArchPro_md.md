
2020/5/20+

PlatformIO cli ArchPro
# PlatformIO cli ArchPro

## 概要
開発ツールPlatformIOを以下のArchProで使う(mbed版)。VisualCodeのプラグインとしてPlatformIOを使用することができるが、ここでは、cliとしての使い方について記する。
(ホストPCとしてはubuntuを想定している)

[Arch Pro](http://akizukidenshi.com/catalog/g/gM-09166/)  

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
interface firmwareをアップデートする：
```
wget https://developer.mbed.org/media/uploads/yihui/lpc11u35_lpc1768_if_mbed_20150128.bin

# ボードのボタンを押しながらホストＰＣとUSB接続する
#「MBED LOADER」のホルダが現れるので、
# ここに上のダウンロードしたファイルをドラッグ＆ドロップする

# ファームウェアの書き込みが開始されるので
# 書き込みが終了するまで待つ

```
なお、以下のバイナリが最新版のようだが、linuxでエラー(容量不足)になって書き込めなかったので、
上で説明したバイナリを利用した：  
https://armmbed.github.io/DAPLink/firmware/0253_lpc11u35_archpro_0x0000.bin  
(windowsでは書き込めるのかもしれない)  

## テスト用プロジェクト blink を作成/実行する
```bash

#ターゲットボードのtarget名を検索する
# (ここでは seeedArchPro を検索する)

$ pio boards | grep Arch
#出力例：
seeedArchBLE       NRF51822  16MHz        128KB    16KB   Seeed Arch BLE
seeedArchLink      NRF51822  16MHz        256KB    16KB   Seeed Arch Link
seeedArchGPRS      LPC11U37          48MHz        128KB    10KB   Seeed Arch GPRS V2
seeedArchPro       LPC1768           96MHz        512KB    64KB   Seeed Arch Pro
seeedArchMax       STM32F407VET6   168MHz       512KB     192KB     Seeed Arch Max
 
#target名として「seeedArchPro」が判明した

# プロジェクトblink のディレクトリを作成する
mkdir blink	
cd blink
# 以下を実行して必要なファイルを作成する
pio init --board seeedArchPro

nano platformio.ini
# platformio.iniを以下のように編集する：
```

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

[env:seeedArchPro]
platform = nxplpc
board = seeedArchPro
framework = mbed

upload_port = /media/USER/MBED
upload_protocol = mbed

debug_tool = cmsis-dap

```
「/media/USER/MBED」の部分は、自分の環境に合わせる。  

続き： 

```bash
# テスト用のmain.cppを作成する
nano src/main.cpp
以下の内容に編集する：
```
```c++

#include "mbed.h"

//DigitalOut  led1(LED4); // ORANGE
DigitalOut  led1(LED3); // BLUE
//DigitalOut  led1(LED2); // RED
//DigitalOut  led1(LED1); // GREEN

int main() {
    while(1) {
        led1 = !led1; 
        wait(0.1); 
    }
}
```
続き：
```bash

# build
pio run

# ボードをホストPCに接続する
# build&upload(flash)
pio run -t upload
# buildしないで書き込む場合は以下を実行する：
pio run -t nobuild -t upload -v
# -v は、詳細を表示するオプション

# 以上で、基本的な操作としては完了となる

```
書き込み後、ボード上にある４つのLEDのうちの１つが点滅する。
VScodeにplatformioのプラグインがインストールされているのであれば  
code ./blink  
でVScodeを起動して、ソース上の行番号の左側をクリックして
ブレークポイントを設定してデバッガができる。  

## 別のサンプル例
```c++


#include "mbed.h"

Serial pc(USBTX, USBRX, 115200); // tx, rx

DigitalIn  mypin(P0_23); // change this to the button on your board
DigitalOut myled(LED3);

int main()
{
    // check mypin object is initialized and connected to a pin
    if(mypin.is_connected()) {
        pc.printf("mypin is connected and initialized! \n\r");
    }
    
    // Optional: set mode as PullUp/PullDown/PullNone/OpenDrain
    mypin.mode(PullNone); 
    
    // press the button and see the console / led change
    while(1) {
        pc.printf("mypin has value : %d \n\r", mypin.read());
        myled = mypin; // toggle led based on value of button
        wait(0.25);
    }
}
```
書き込みが完了すると、以下の動作になる：   
P0_23が入力ポートになっているので、このポートをGNDか3V3に接続すると、
値か0か1に変化する。この値が0の場合、青いLED3が光る。入力ポートの
状態は、シリアル(/dev/ttyACM0)出力される。


## 参考情報

[Seeeduino-Arch-Pro mbed](https://os.mbed.com/platforms/Seeeduino-Arch-Pro/)  
[mbed/Platform/ArchPro wiki](https://wiki.seeedstudio.com/Arch_Pro/)   

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

[ARMmbed/DAPLink](https://github.com/ARMmbed/DAPLink)  


以上

