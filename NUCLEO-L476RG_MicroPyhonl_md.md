
2020/2/7

Nucleo L476RG MicroPython Install
# Nucleo L476RG MicroPython Install

## 概要
「Nucleo L476RG」にMicroPythonをインストールする方法について記載する
(ホストPCとしてはubuntuを想定している)

## 事前準備
(1)ampyのインストール   
AMPY_PORTは、自分の環境に合わせる。
```bash

pip install adafruit-ampy
export AMPY_PORT=/dev/ttyACM0
export AMPY_BAUD=115200
```
(2)picocomのインストール
```bash

sudo apt-get install picocom
```
以上のうち、exportしているものは、.bashrcに登録することを勧める。


## 参照URL
[The MicroPython project](https://github.com/micropython/micropython)     
[ＳＴＭ３２　Ｎｕｃｌｅｏ　Ｂｏａｒｄ　ＳＴＭ３２Ｌ４７６ＲＧ](http://akizukidenshi.com/catalog/g/gM-10177/)   
[Quick reference for the pyboard](http://docs.micropython.org/en/latest/pyboard/quickref.html)
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)    
[MicroPython pyboard v1.1](https://www.switch-science.com/catalog/3488/)   
[新しいボードへMicroPythonを対応させる方法をざっくり解説](https://blog.boochow.com/article/mp-stm-nucleo-l432kc.html)   



## ビルド手順
```

mkdir mp
cd mp

git clone https://github.com/micropython/micropython.git

cd micropython
cd mpy-cross
make

cd ports/stm32 
make submodules
make BOARD=NUCLEO_L476RG

```
この時点でfirmwareがビルドされる。

## ビルドしたファームウェアの書き込み
以下のコマンドでファームウェア書き込みができるはずだが..   
dfu-util -a 0 -D ./build-NUCLEO_F446RE/firmware.dfu   
「dfu-util: No DFU capable USB device available」のエラーが出て
書き込めなかった。   
(たぶん、今回のボードがDFU対応になっていないせい)   
なので、PlatformIOのツール(stlink)を使う。   
新規プロジェクトでL476RGのプロジェクト名で   
board:「ST Nucleo L476RG」  
platform: mbed  
に設定する。   
(書き込み方法はデフォルトの「upload_protocol = stlink」のままなので何も設定しない)   

~/Documents/PlatformIO/Projects/L476RG/.pio/build/nucleo_l476rg/firmware.elf  
を今回ビルドした以下のelfに差し替える。   
./build-NUCLEO_L476RG/firmware.elf


実行例：
```bash

cp ./build-NUCLEO_L476RG/firmware.elf  ~/Documents/PlatformIO/Projects/L476RG/.pio/build/nucleo_l476rg/firmware.elf  
```

差し替えた後、PlatformIOのupload（書き込み）を実行する。

以下、書き込み時のログ出力例：
```bash

> Executing task in folder L476RG: platformio run --target upload <

Processing nucleo_l476rg (platform: ststm32; board: nucleo_l476rg; framework: mbed)
---------------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/ststm32/nucleo_l476rg.html
PLATFORM: ST STM32 6.0.0 > ST Nucleo L476RG
HARDWARE: STM32L476RGT6 80MHz, 128KB RAM, 1MB Flash
DEBUG: Current (stlink) On-board (stlink) External (blackmagic, jlink)
PACKAGES: tool-dfuutil 1.9.190708, toolchain-gccarmnoneeabi 1.70201.0 (7.2.1), framework-mbed 5.51401.191023 (5.14.1), tool-stm32duino 1.0.1, tool-openocd 2.1000.190707 (10.0)
Collecting mbed sources...
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 14 compatible libraries
Scanning dependencies...
No dependencies
Building in release mode
Checking size .pio/build/nucleo_l476rg/firmware.elf
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
DATA:    [=         ]   6.9% (used 8996 bytes from 131072 bytes)
PROGRAM: [===       ]  31.9% (used 334456 bytes from 1048576 bytes)
Configuring upload protocol...
AVAILABLE: blackmagic, jlink, mbed, stlink
CURRENT: upload_protocol = stlink
Uploading .pio/build/nucleo_l476rg/firmware.elf
xPack OpenOCD, 64-bit Open On-Chip Debugger 0.10.0+dev (2019-07-17-11:25)
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
debug_level: 1

srst_only separate srst_nogate srst_open_drain connect_deassert_srst

target halted due to debug-request, current mode: Thread 
xPSR: 0x01000000 pc: 0x080013d0 msp: 0x10001288
** Programming Started **
Warn : block write succeeded
** Programming Finished **
** Verify Started **
** Verified OK **
** Resetting Target **
shutdown command invoked

===== [SUCCESS] Took 20.89 seconds =====

Terminal will be reused by tasks, press any key to close it.

```
以上でfirmwareがボードに書き込まれる。


## 動作確認
picocomを使いボードとシリアルで通信する。
以下、通信例：
```bash

$ picocom /dev/ttyACM0 -b115200
#反応がない場合、Ctrl-C,Ctrl-Dを押す。
>>> 
MPY: sync filesystems
MPY: soft reboot
MicroPython v1.12-58-g7ef2f65-dirty on 2020-02-07; NUCLEO-L476RG with STM32L476RG
Type "help()" for more information.
>>> 

>>> import os
>>> os.uname()
(sysname='pyboard', nodename='pyboard', release='1.12.0', version='v1.12-58-g7ef2f65-dirty on 2020-02-07', machine='NUCLEO-L476RG with STM32L476RG')
>>> 

>>> import gc
>>> gc.collect()
>>> gc.mem_free()
85968
>>> 

>>> list(5 * x + y for x in range(10) for y in [4, 2, 1])
[4, 2, 1, 9, 7, 6, 14, 12, 11, 19, 17, 16, 24, 22, 21, 29, 27, 26, 34, 32, 31, 39, 37, 36, 44, 42, 41, 49, 47, 46]
>>> 


```

内蔵モジュール表示：
```bash

>>> help('modules')
__main__          math              ucollections      urandom
_onewire          micropython       uctypes           ure
builtins          network           uerrno            uselect
cmath             onewire           uhashlib          usocket
dht               pyb               uheapq            ustruct
framebuf          stm               uio               utime
gc                sys               ujson             utimeq
lcd160cr          uarray            umachine          uzlib
lcd160cr_test     ubinascii         uos
Plus any modules on the filesystem
>>> 

```

## ampy実行例
```bash

$ ampy ls /flash
/flash/README.txt
/flash/boot.py
/flash/main.py
/flash/pybcdc.inf

$ ampy get /flash/README.txt
This is a MicroPython board

You can get started right away by writing your Python code in 'main.py'.

For a serial prompt:
 - Windows: you need to go to 'Device manager', right click on the unknown device,
   then update the driver software, using the 'pybcdc.inf' file found on this drive.
   Then use a terminal program like Hyperterminal or putty.
 - Mac OS X: use the command: screen /dev/tty.usbmodem*
 - Linux: use the command: screen /dev/ttyACM0

Please visit http://micropython.org/help/ for further help.

$ ampy get /flash/boot.py
# boot.py -- run on boot-up
# can run arbitrary Python, but best to keep it minimal

import machine
import pyb
pyb.country('US') # ISO 3166-1 Alpha-2 code, eg US, GB, DE, AU
#pyb.main('main.py') # main script to run after this one
#pyb.usb_mode('VCP+MSC') # act as a serial and a storage device
#pyb.usb_mode('VCP+HID') # act as a serial device and a mouse

$ ampy get /flash/main.py
# main.py -- put your code here!

```

## サンプル・スクリプト
blink.py
```python

print("LED blinking test...")
led = pyb.LED(1) # (pin_A5) // Green LED on Nucleo

while True:
    led.on()
    pyb.delay(100)
    led.off()
    pyb.delay(100)

```
cpuが同じこともあり、pyboardと同じボードと見做されているようだ。


実行例：
```bash

$ ampy run blink.py 


```
この場合、flashにスクリプトを書き込まずに直接RAMで実行される。


## スクリプトを保存/実行する場合
ボードの電源オンで直接スクリプトを実行する場合は
/flash/main.pyにプログラムを置く。

実行例：
```bash

$ ampy put blink.py /flash/main.py 


```
いったん[reset]を押すと書き込んだプログラムが起動する。

##  ピン・アサイン状況
ソースの./micropython/ports/stm32/boards/NUCLEO_L476RG/mpconfigboard.h
を見るとピンのアサイン状況が分かる：   
以下、抜粋：
```c++


// I2C busses
#define MICROPY_HW_I2C1_SCL (pin_B8)
#define MICROPY_HW_I2C1_SDA (pin_B9)
...
// SPI busses
#define MICROPY_HW_SPI1_NSS     (pin_A4)
#define MICROPY_HW_SPI1_SCK     (pin_B3)
#define MICROPY_HW_SPI1_MISO    (pin_B4)
#define MICROPY_HW_SPI1_MOSI    (pin_B5)
#define MICROPY_HW_SPI2_NSS     (pin_B12)
#define MICROPY_HW_SPI2_SCK     (pin_B13)
#define MICROPY_HW_SPI2_MISO    (pin_B14)
#define MICROPY_HW_SPI2_MOSI    (pin_B15)
...
// LEDs
#define MICROPY_HW_LED1 (pin_A5) // Green LED on Nucleo

```

## 技術情報

[NUCLEO-L476RG](https://os.mbed.com/platforms/ST-Nucleo-L476RG/)  
[Quick reference for the pyboard](https://docs.micropython.org/en/latest/pyboard/quickref.html)   

## WSL(Windows Subsystem for Linux)環境での留意点
windows側のCOM_Port番号xがlinux側の/dev/ttySxにマッピングされる。
たとえば、デバイスのポートが COM3 の時、WSLでは /dev/ttyS3 になる。
そのままだと、sudoを付けないとアクセスできないので、
以下のコマンドで権限を変更する：

sudo chmod 666 /dev/ttyS3

当然のことだが、windows側でserialドライバーのインストールが完了していることが前提になる。

WSLでビルドを実際にはやっていないが、たぶん、ビルドまでは、成功すると思われる。
書き込みツールのstlinkは、WSL側では、動作しないと思われるが、ビルドしたfirmwareを
windows側にコピーして、windowsのstlinkで書き込むことはできると思われる。

## 追記情報(DFU-mode)
後日、調べたら、以下のようにDFU-modeにする方法が分かった。  

BOOT0端子をH(3.3V)に接続し、ボートをリセット(電源オン)すると、PA12,PA11(D+,D-)がUSB端子になり、DFU-modeになる。なので、そこにホストPCのUSBを接続し、ボード立ち上げ後、dfu-utilを使用すればファームウェアを書き込むことができる。   
ただし、色々接続しなければならないので、本件のボードの場合、DFU-modeではなく、stlinkで書き込むのが正解だと思う。   

以下、DFU-modeに関する詳細情報：   
[STM32 NUCLEO AND DFU USB BOOTLOADING](https://mobilewill.us/stm32-nucleo-and-dfu-usb-bootloading/)   
・接続時の写真があり分かりやすい。   
[STM32シリーズへの書き込み(DFU編)](https://rur.mech.tuat.ac.jp/robotbot/archives/645)   
[USB Tester 3.0](https://www.tindie.com/products/friedcircuits/usb-tester-30/)   

以上
