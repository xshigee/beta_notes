
2020/1/19+

NUCLEO F446RE MicroPython Install
# NUCLEO F446RE MicroPython Install

## 概要
「NUCLEO F446RE」にMicropythonをインストールする方法について記載する
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
[ＳＴＭ３２　Ｎｕｃｌｅｏ　Ｂｏａｒｄ　ＳＴＭ３２Ｆ４４６ＲＥ](http://akizukidenshi.com/catalog/g/gM-10176/)   
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
make BOARD=NUCLEO_F446RE

# ここでは直接関係ないが、他に以下のボードがビルドできた
#make BOARD=STM32F4DISC
#make BOARD=NUCLEO_F401RE
#make BOARD=NUCLEO_F411RE
#make BOARD=NUCLEO_L476RG
# 以下はerrorになった
#make BOARD=NUCLEO_F767ZI

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
./build-NUCLEO_F446RE/firmware.elf  
   
差し替えた後、PlatformIOのupload（書き込み）を実行する。

以下、書き込み時のログ出力例：
```bash


> Executing task in folder F446RE_mbed_01: platformio run --target upload <

Processing nucleo_f446re (platform: ststm32; board: nucleo_f446re; framework: mbed)
------------------------------------------------------------
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
DATA:    [          ]   3.1% (used 4000 bytes from 131072 bytes)
PROGRAM: [=         ]   9.3% (used 48956 bytes from 524288 bytes)
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
xPSR: 0x01000000 pc: 0x08001cf8 msp: 0x20020000
** Programming Started **
** Programming Finished **
** Verify Started **
** Verified OK **
** Resetting Target **
shutdown command invoked
=====[SUCCESS] Took 11.41 seconds =====

Terminal will be reused by tasks, press any key to close it.

```
以上でfirmwareがボードに書き込まれる。

## 動作確認
picocomを使いボードとシリアルで通信する。
以下、通信例：
```bash

$ picocom /dev/ttyACM0 -b115200
>>> import os
>>> os.uname()
(sysname='pyboard', nodename='pyboard', release='1.12.0', version='v1.12-68-g3032ae1 on 2020-01-19', machine='NUCLEO-F446RE with STM32F446xx')
>>> list(5 * x + y for x in range(10) for y in [4, 2, 1])
[4, 2, 1, 9, 7, 6, 14, 12, 11, 19, 17, 16, 24, 22, 21, 29, 27, 26, 34, 32, 31, 39, 37, 36, 44, 42, 41, 49, 47, 46]
>>> 
```

オンライン・ヘルプ表示：
```bash

>>> help()
Welcome to MicroPython!

For online help please visit http://micropython.org/help/.

Quick overview of commands for the board:
  pyb.info()    -- print some general information
  pyb.delay(n)  -- wait for n milliseconds
  pyb.millis()  -- get number of milliseconds since hard reset
  pyb.Switch()  -- create a switch object
                   Switch methods: (), callback(f)
  pyb.LED(n)    -- create an LED object for LED n (n=1,2,3,4)
                   LED methods: on(), off(), toggle(), intensity(<n>)
  pyb.Pin(pin)  -- get a pin, eg pyb.Pin('X1')
  pyb.Pin(pin, m, [p]) -- get a pin and configure it for IO mode m, pull mode p
                   Pin methods: init(..), value([v]), high(), low()
  pyb.ExtInt(pin, m, p, callback) -- create an external interrupt object
  pyb.ADC(pin)  -- make an analog object from a pin
                   ADC methods: read(), read_timed(buf, freq)
  pyb.DAC(port) -- make a DAC object
                   DAC methods: triangle(freq), write(n), write_timed(buf, freq)
  pyb.RTC()     -- make an RTC object; methods: datetime([val])
  pyb.rng()     -- get a 30-bit hardware random number
  pyb.Servo(n)  -- create Servo object for servo n (n=1,2,3,4)
                   Servo methods: calibration(..), angle([x, [t]]), speed([x, [t]])
  pyb.Accel()   -- create an Accelerometer object
                   Accelerometer methods: x(), y(), z(), tilt(), filtered_xyz()

Pins are numbered X1-X12, X17-X22, Y1-Y12, or by their MCU name
Pin IO modes are: pyb.Pin.IN, pyb.Pin.OUT_PP, pyb.Pin.OUT_OD
Pin pull modes are: pyb.Pin.PULL_NONE, pyb.Pin.PULL_UP, pyb.Pin.PULL_DOWN
Additional serial bus objects: pyb.I2C(n), pyb.SPI(n), pyb.UART(n)

Control commands:
  CTRL-A        -- on a blank line, enter raw REPL mode
  CTRL-B        -- on a blank line, enter normal REPL mode
  CTRL-C        -- interrupt a running program
  CTRL-D        -- on a blank line, do a soft reset of the board
  CTRL-E        -- on a blank line, enter paste mode

For further help on a specific object, type help(obj)
For a list of available modules, type help('modules')

>>> 
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

$ ampy -p /dev/ttyACM0 ls
/flash

$ ampy -p /dev/ttyACM0 ls /flash
/flash/boot.py
/flash/main.py

ampy -p /dev/ttyACM0 get /flash/boot.py
# boot.py -- run on boot-up
# can run arbitrary Python, but best to keep it minimal

import machine
import pyb
pyb.country('US') # ISO 3166-1 Alpha-2 code, eg US, GB, DE, AU
#pyb.main('main.py') # main script to run after this one

$ ampy -p /dev/ttyACM0 get /flash/main.py
# main.py -- put your code here!

```

## サンプルスクリプト
blink.py
```python

while True:
# LED(1) // Green LED on board
    pyb.LED(1).on()
    pyb.delay(1000)
    pyb.LED(1).off()
    pyb.delay(250)
```
cpuが同じこともあり、pyboardと同じボードと見做されているようだ。

実行例：
```bash

$ ampy -p /dev/ttyACM0 run blink.py 


```
この場合、flashにスクリプトを書き込まずに直接RAMで実行される。

ボードの電源オンで直接スクリプトを実行する場合は   
以下のようにする。(blink.pyをmain.pyに上書きする)
```bash

$ ampy -p /dev/ttyACM0 put blink.py /flash/main.py

```
この書き込み後、リセットボタンを押すか、またはUSBコネクタを抜き差しすると、自動的にスクリプトが動作する。 


## 備考
本件のボードが2000円弱なのに対して、pyboardが7000円強するようなので、今回の方法でpyboard化できるので、お買い得かもしれない。

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



