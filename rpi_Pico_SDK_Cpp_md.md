
2021/2/23+   
openocdのudev登録を追加。  
gdbの実行例を追加。

2021/2/17  
必要なライブラリのインストールの追加。  

2021/2/4  
初版

rpi Pico SDK C/C++
# rpi Pico SDK C/C++

## 概要
以下のPicoボードにPico_SDKでC言語をビルドする。  
(ホストPCとしてはubuntuを想定している)

・[Raspberry Pi Pico](https://www.switch-science.com/catalog/6900/)  


## 事前準備
以下の手順でツールをインストールする：
```

sudo apt update
sudo apt install gcc-arm-none-eabi build-essential
sudo apt install curl git picocom

# udevのrulesを登録する
curl -fsSL https://raw.githubusercontent.com/platformio/platformio-core/master/scripts/99-platformio-udev.rules | sudo tee /etc/udev/rules.d/99-platformio-udev.rules

sudo udevadm control --reload-rules

sudo usermod -a -G dialout $USER
sudo usermod -a -G plugdev $USER

# 必要なライブラリのインストール
sudo apt install libusb-dev
sudo apt install libusb-1.0-0-dev

# ビルド時にエラーになったのでcmake最新版をインストールする
# 古いcmakeを削除する
sudo apt remove cmake

mkdir ~/cmake_latest
cd ~/cmake_latest

wget https://github.com/Kitware/CMake/releases/download/v3.17.1/cmake-3.17.1.tar.gz
tar zxvf cmake-3.17.1.tar.gz

cd cmake-3.17.1/
./bootstrap
make
sudo make install

# gdbのインストール
sudo apt install gdb-multiarch
```

## Pico_SDKの準備
以下の手順で必要なものをインストールする：
```

# pico SDK のダウンロード
mkdir ~/pico
git clone -b master https://github.com/raspberrypi/pico-sdk.git
cd pico-sdk
git submodule update --init

# picotoolのインストール
cd ~/pico
git clone https://github.com/raspberrypi/picotool.git
cd picotool
mkdir build
cd build
export PICO_SDK_PATH=../../pico-sdk
cmake ..
make

sudo cp picotool  /usr/local/bin/

# elf2uf2のインストール
# (sdkの環境のまま使用するときは不要)
cd ~/pico
cd pico-sdk/tools/elf2uf2
mkdir build
cd build
export PICO_SDK_PATH=../../../../pico-sdk
cmake ..
make

sudo cp elf2uf2  /usr/local/bin/

# pioasmのインストール
# (sdkの環境のまま使用するときは不要)
cd ~/pico
cd pico-sdk/tools/pioasm
mkdir build
cd build
export PICO_SDK_PATH=../../../../pico-sdk
cmake ..
make

sudo cp pioasm  /usr/local/bin/
```
以下を.bashrcを登録する：
```
export PICO_SDK_PATH=~/pico/pico-sdk
```

## サンプルプログラムのダウンロード
以下の手順でダウンロードする：
```

cd ~/pico
git clone -b master https://github.com/raspberrypi/pico-examples.git
```

## サンプルプログラム(blink)のビルド

```

cd ~/pico
cd pico-examples
mkdir build
cd build
export PICO_SDK_PATH=../../pico-sdk
cmake ..
cd blink
make -j4

# ビルド結果を確認する
ls
CMakeFiles  blink.bin  blink.elf      blink.hex  cmake_install.cmake
Makefile    blink.dis  blink.elf.map  blink.uf2  elf2uf2

```

## 別のサンプル(hello_workd)のビルド

```
cd ~/pico
cd pico-examples
# すでにあるbuildをいったん削除する
rm -rf build/
mkdir build
cd build
export PICO_SDK_PATH=../../pico-sdk
cmake ..
cd hello_world
make -j4

# この例では２つのビルド結果ができるので確認する
# その１：
ls serial/
CMakeFiles           hello_serial.bin  hello_serial.elf.map
Makefile             hello_serial.dis  hello_serial.hex
cmake_install.cmake  hello_serial.elf  hello_serial.uf2

# その２：
ls usb/
CMakeFiles           hello_usb.bin  hello_usb.elf.map
Makefile             hello_usb.dis  hello_usb.hex
cmake_install.cmake  hello_usb.elf  hello_usb.uf2

```
他のサンプルも上の手順で「cd hello_world」のところをビルドしたい
サンプルのディレクトリに置き換えて実行すればビルドできる。

## ビルドしたファームウェアの確認
以下のやり方でファームウェアの内容を確認できる：
```

$ picotool info firmware.uf2
File firmware.uf2:

Program Information
 name:            MicroPython
 version:         de1239b6a
 features:        USB REPL
                  thread support
 frozen modules:  _boot, rp2, ds18x20, onewire, uasyncio, uasyncio/core,
                  uasyncio/event, uasyncio/funcs, uasyncio/lock, uasyncio/stream
```

## ビルドしたファームウェアの書き込み
以下の手順で書き込む：(ディレクトリ位置は例)
```

# BOOTSELボタンを押しながらPicoボードをホストPCにUSB接続する。
# BOOTSELボタンを離す。
# 書き込み用のRPI-RP2ディレクトリが現れる

cd build
cd hello_world
sudo cp usb/hello_usb.uf2 /media/<USER>/RPI-RP2/
```
以上でファームウェア(.uf2)がPicoボードに書き込まれる。  
(\<USER\>の部分はホストの環境に合わせる)

## 動作確認
picocomを使いPicoボードとPC間でUSBシリアルで通信する。  
通信例：
```bash

$ picocom  -b115200 /dev/ttyACM0
<省略>
Hello, world!
Hello, world!
Hello, world!
Hello, world!
Hello, world!
Hello, world!
Hello, world!
Hello, world!
<省略>
```

## プロジェクトを自動生成する
プログラムの雛形を自動生成できるので使ってみる。   
以下の手順でgeneratorを動かす：
```

cd pico
git clone https://github.com/raspberrypi/pico-project-generator.git
cd pico-project-generator
./pico_project.py --gui
```
以上でgeneratorのGUIが起動する。

以下のようにパラメータを入力する：
```

(2)Project Name:
任意のプログラムを入力する(例：mycode)

(2)Location:
プロジェクトを置くディレクトリを指定する(例：~/pico_ws)

(3)Feature:
利用する機能にチェックを入れる

(4)Console Option:
arduinoボード同様にコンソールをUSBシリアルにしたい場合、以下を選択する：
[Console over USB]

(5)Code Option:
そのままで良い。

(6)Build Option:
そのままで良い。

(7)パラメータを入力したら
[OK]をクリックするとプロジェクトが生成される。

(3)generatorから抜けるとき
[Quit]をクリックする
```

ビルド/書き込み
以下の手順でビルドできる：(例)
```

cd ~/pico_ws/mycode
cd build
make
cp mycode.uf2 /media/<USER>/RPI-RP2
```
この場合、ソースは~/pico_ws/mycodeにある。

実際には、ここにある雛形のソース(mycode.c)を編集してプログラムする。

## picoprobe
PICOボードをデバッグ(picoprobe)ボードにするためのファームウェアが提供されているので、それをビルドする：

build
```

export PICO_SDK_PATH=~/pico/pico-sdk
cd ~/pico
git clone https://github.com/raspberrypi/picoprobe.git
cd picoprobe
mkdir build
cd build
cmake ..
make -j4

# PICOボードに書き込む
sudo cp  picoprobe.uf2 /media/<USER>/RPI-RP2/
```
以上で、PICOボードが以下の機能をもつデバッグボードになる。
1. CDC UART (USBシリアル)  
2. SWD probe  

picoprobe対応のOpenOCDのビルド
```

cd ~/pico
sudo apt install automake autoconf build-essential texinfo libtool libftdi-dev libusb-1.0-0-dev
git clone https://github.com/raspberrypi/openocd.git --branch picoprobe --depth=1
cd openocd
./bootstrap
./configure --enable-picoprobe
make -j4
sudo make install
```

openocdのudev登録   
以下の手順で登録する：
```

echo 'ATTRS{idVendor}=="2e8a", ATTRS{idProduct}=="0004", MODE="0666"' | sudo tee /etc/udev/rules.d/99-picoprobe.rules
sudo udevadm trigger
```
この登録がないと一般ユーザーとしてのopenocdを起動した場合「Error: libusb_open() failed with LIBUSB_ERROR_ACCESS」のエラーが出る。


起動方法
```

openocd -f interface/picoprobe.cfg -f target/rp2040.cfg -s tcl

Open On-Chip Debugger 0.10.0+dev-g14c0d0d-dirty (2021-02-01-08:41)
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
Info : only one transport option; autoselect 'swd'
Warn : Transport "swd" was already selected
adapter speed: 5000 kHz

Info : Hardware thread awareness created
Info : Hardware thread awareness created
Info : RP2040 Flash Bank Command
Info : Listening on port 6666 for tcl connections
Info : Listening on port 4444 for telnet connections
Info : clock speed 5000 kHz
Info : DAP init failed 
# ターゲットボードを接続していないのでDAPでエラーになっている

# Connect GDB as you usually would with
# target remote localhost:3333 
```
ドキュメントの起動方法はミス・スペルがあったので注意のこと。

実際にターゲットボードと接続したときのgdbでの実行例
```

#別のterminalで以下を実行してopenocdを起動する(サーバーとして動作する)
openocd -f interface/picoprobe.cfg -f target/rp2040.cfg -s tcl

#もう一つのterminalで以下を実行する：
$ cd ~/pico/pico-examples/build/blink
$ gdb-multiarch blink.elf
GNU gdb (Ubuntu 9.2-0ubuntu1~20.04) 9.2
Copyright (C) 2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from blink.elf...
# 以下でopenocdと接続する
(gdb) target remote :3333
Remote debugging using :3333
0x100009ce in sleep_ms ()
(gdb) mon reset init
target halted due to debug-request, current mode: Thread 
xPSR: 0xf1000000 pc: 0x000000ee msp: 0x20041f00
target halted due to debug-request, current mode: Thread 
xPSR: 0xf1000000 pc: 0x000000ee msp: 0x20041f00
(gdb) load
Loading section .boot2, size 0x100 lma 0x10000000
Loading section .text, size 0x2608 lma 0x10000100
Loading section .rodata, size 0xd8 lma 0x10002708
Loading section .binary_info, size 0x1c lma 0x100027e0
Loading section .data, size 0x99c lma 0x100027fc
Start address 0x10000104, load size 12696
Transfer rate: 7 KB/sec, 2539 bytes/write.
# mainにブレークポイントを設定する
(gdb) b main
Breakpoint 1 at 0x1000037c
# 実行する
(gdb) c
Continuing.
Note: automatically using hardware breakpoints for read-only addresses.
target halted due to debug-request, current mode: Thread 
xPSR: 0x01000000 pc: 0x00000178 msp: 0x20041f00

# 以下のようにブレークする
Thread 1 hit Breakpoint 1, 0x1000037c in main ()
# さらに実行する
(gdb) c
Continuing.
^Ctarget halted due to debug-request, current mode: Thread 
xPSR: 0x01000000 pc: 0x00000178 msp: 0x20041f00

Thread 1 received signal SIGINT, Interrupt.
0x100009ce in sleep_ms ()
<省略>
```
なお、targetボードには、あらかじめblink.uf2を書き込んでおく。
[]()  
[]()  
DebugボードとTargetボードの接続方法：  
(1)DebuggerボードUSBコネクタ側 上面左側
| Pin | Target側Pin |
| ---: | :--- |
|GP0| NC |
|GP1| NC |
|GND| GND |
|GP2| SWCLK |
|GP3| SWDIO |
|GP4(TX)| GP1(RX) |
|GP5(RX)| GP0(TX)|
[]()   
[]()  

(2)DebuggerボードUSBコネクタ側 上面右側
| Pin | Target側Pin |
| ---: | :--- |
|VBUS(5V)|NC|
|VSYS(IN 1.8V-5.5V)| NC |
|GND| GND |
|3V3EN| NC |
|3V3(OUT)| VSYS |
[]()  
[]()  

(1) NC: None Connection   
(2) VBUS(5V):  
・USBコネクタの5Vに接続されているのでUSBコネクタで給電している場合、5Vが出力される。  
・USBホストモードで使用する際は、ここに5Vを給電する。  
(3) 配線状況：  
TargetボードはDebugボードから給電される配線になっている。   
したがって、USBコネクタとPCを接続するのはDebugボードのみになる。

## 参照情報


・[How to add a reset button to your Raspberry Pi Pico](https://www.raspberrypi.org/blog/how-to-add-a-reset-button-to-your-raspberry-pi-pico/)   
All you need to do is to wire the GND and RUN pins together and add an extra momentary contact button to your breadboard. Pushing the button will reset the board.

・[Programmable I/O with Raspberry Pi Pico](https://www.seeedstudio.com/blog/2021/01/25/programmable-io-with-raspberry-pi-pico/)  


・[Raspberry Pi Picoスタートガイド C/C++](https://pico.raspberrypi.org/files/getting_started.pdf)   
・[Pico C/C++ SDK ](https://pico.raspberrypi.org/files/pico_sdk.pdf)  
・[Pico SDK docs](https://raspberrypi.github.io/pico-sdk-doxygen/index.html)  


・[Raspberry Pi Pico Datasheet](https://pico.raspberrypi.org/files/pico_datasheet.pdf)  
・[RP2040 Datasheet](https://pico.raspberrypi.org/files/rp2040_datasheet.pdf)  


・[Pico Pinout](https://akizukidenshi.com/download/ds/raspberry/Pico-R3-A4-Pinout.pdf)  


・[Ubuntu 18.04 に Cmake の Latest Release をインストールする](https://qiita.com/comachi/items/d0c1ce5d7b90fe30fced)  

・[Ubuntu 20.04のPCでRaspberry Pi Picoを2台使ってOpenOCDによるデバッグ](https://mickey-happygolucky.hatenablog.com/entry/2021/02/18/235625)   


以上
