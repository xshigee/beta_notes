
2020/2/6

Nucleo F401RE MicroPython Install
# Nucleo F401RE MicroPython Install

## 概要
「Nucleo F401RE」にMicropythonをインストールする方法について記載する
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
[ＳＴＭ３２　Ｎｕｃｌｅｏ　Ｂｏａｒｄ　ＳＴＭ３２Ｆ４０１](http://akizukidenshi.com/catalog/g/gM-07723/)  
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
make BOARD=NUCLEO_F401RE

```
この時点でfirmwareがビルドされる。

## ビルドしたファームウェアの書き込み
以下のコマンドでファームウェア書き込みができるはずだが..   
dfu-util -a 0 -D ./build-NUCLEO_F446RE/firmware.dfu   
「dfu-util: No DFU capable USB device available」のエラーが出て
書き込めなかった。   
(たぶん、今回のボードがDFU対応になっていないせい)   
なので、PlatformIOのツール(stlink)を使う。   
新規プロジェクトでF446RE_mbed_01のプロジェクト名で   
board:「ST Nucleo F446RE」  
platform: mbed  
に設定する。   
(書き込み方法はデフォルトの「upload_protocol = stlink」のままなので何も設定しない)   

~/Documents/PlatformIO/Projects/F446RE_mbed_01/.pio/build/nucleo_f446re/firmware.elf  
を今回ビルドした以下のelfに差し替える。   
./build-NUCLEO_F401RE/firmware.elf

実行例：
```bash

cp ./build-NUCLEO_F401RE/firmware.elf ~/Documents/PlatformIO/Projects/F446RE_mbed_01/.pio/build/nucleo_f446re/																																																																																																																																																																																																																firmware.elf  
```

差し替えた後、PlatformIOのupload（書き込み）を実行する。

以下、書き込み時のログ出力例：
```bash

> Executing task in folder F446RE_mbed_01: platformio run --target upload <

Processing nucleo_f446re (platform: ststm32; board: nucleo_f446re; framework: mbed)
----------------------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/ststm32/nucleo_f446re.html
PLATFORM: ST STM32 6.0.0 > ST Nucleo F446RE
HARDWARE: STM32F446RET6 180MHz, 128KB RAM, 512KB Flash
DEBUG: Current (stlink) On-board (stlink) External (blackmagic, jlink)
PACKAGES: tool-dfuutil 1.9.190708, toolchain-gccarmnoneeabi 1.70201.0 (7.2.1), framework-mbed 5.51401.191023 (5.14.1), tool-stm32duino 1.0.1, tool-openocd 2.1000.190707 (10.0)
Collecting mbed sources...
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 14 compatible libraries
Scanning dependencies...
No dependencies
Building in release mode
Checking size .pio/build/nucleo_f446re/firmware.elf
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
DATA:    [=         ]  14.4% (used 18936 bytes from 131072 bytes)
PROGRAM: [=====     ]  53.6% (used 280908 bytes from 524288 bytes)
Configuring upload protocol...
AVAILABLE: blackmagic, jlink, mbed, stlink
CURRENT: upload_protocol = stlink
Uploading .pio/build/nucleo_f446re/firmware.elf
xPack OpenOCD, 64-bit Open On-Chip Debugger 0.10.0+dev (2019-07-17-11:25)
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
debug_level: 1

srst_only separate srst_nogate srst_open_drain connect_deassert_srst

target halted due to debug-request, current mode: Thread 
xPSR: 0x01000000 pc: 0x08002364 msp: 0x20002160
** Programming Started **
** Programming Finished **
** Verify Started **
** Verified OK **
** Resetting Target **
shutdown command invoked

===== [SUCCESS] Took 23.76 seconds =====

Terminal will be reused by tasks, press any key to close it.

```
以上でfirmwareがボードに書き込まれる。


## 動作確認
picocomを使いボードとシリアルで通信する。
以下、通信例：
```bash

$ picocom /dev/ttyACM0 -b115200
#反応がない場合、Ctrl-C,Ctrl-Dを押す。
PY: sync filesystems
MPY: soft reboot
MicroPython v1.12-58-g7ef2f65-dirty on 2020-02-06; NUCLEO-F401RE with STM32F401xE
Type "help()" for more information.
>>> 

>>> import os
>>> os.uname()
(sysname='pyboard', nodename='pyboard', release='1.12.0', version='v1.12-58-g7ef2f65-dirty on 2020-02-06', machine='NUCLEO-F401RE with STM32F401xE')
>>> 

>>> import gc
>>> gc.collect()
>>> gc.mem_free()
60240
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
/flash/boot.py
/flash/main.py

$ ampy get /flash/boot.py
# boot.py -- run on boot-up
# can run arbitrary Python, but best to keep it minimal

import machine
import pyb
pyb.country('US') # ISO 3166-1 Alpha-2 code, eg US, GB, DE, AU
#pyb.main('main.py') # main script to run after this one

$ ampy get /flash/main.py
# main.py -- put your code here!


```

## サンプル・スクリプト
blink.py
```python

print("LED blinking test...")
led = pyb.LED(1) # (pin_A5) // Green LD2 LED on Nucleo

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
ソースの./STM32_micropython/ports/stm32/boards/NUCLEO_F401RE/mpconfigboard.h
を見るとピンのアサイン状況が分かる：   
以下、抜粋：
```c++

// I2C busses
#define MICROPY_HW_I2C1_SCL (pin_B8) // Arduino D15, pin 3 on CN10
#define MICROPY_HW_I2C1_SDA (pin_B9) //         D14, pin 5 on CN10
...
// SPI busses
...
#define MICROPY_HW_SPI3_NSS (pin_A4) // Arduino A2,  pin 32 on CN7
#define MICROPY_HW_SPI3_SCK (pin_B3) // Arduino D3,  pin 31 on CN10
#define MICROPY_HW_SPI3_MISO (pin_B4) // Arduino D5,  pin 27 on CN10
#define MICROPY_HW_SPI3_MOSI (pin_B5) // Arduino D4,  pin 29 on CN10
...
// LEDs
#define MICROPY_HW_LED1 (pin_A5) // Green LD2 LED on Nucleo

```


## 技術情報

[NUCLEO-F401RE](https://os.mbed.com/platforms/ST-Nucleo-F401RE/)   
[Quick reference for the pyboard](https://docs.micropython.org/en/latest/pyboard/quickref.html)   


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
