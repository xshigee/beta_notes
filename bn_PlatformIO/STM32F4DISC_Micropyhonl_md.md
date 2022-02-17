
2020/2/1

STM32F4-Discovery MicroPython Install
# STM32F4-Discovery MicroPython Install

## 概要
「STM32F4-Discovery」にMicropythonをインストールする方法について記載する
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
[ＳＴＭ３２Ｆ４ＤＩＳＣＯＶＥＲＹ](http://akizukidenshi.com/catalog/g/gM-05313/)   
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
make BOARD=STM32F4DISC

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
./build-STM32F4DISC/firmware.elf 

実行例：
```bash

cp ./build-STM32F4DISC/firmware.elf ~/Documents/PlatformIO/Projects/F446RE_mbed_01/.pio/build/nucleo_f446re/firmware.elf  
```

差し替えた後、PlatformIOのupload（書き込み）を実行する。

以下、書き込み時のログ出力例：
```bash


> Executing task in folder F446RE_mbed_01: platformio run --target upload <

Processing nucleo_f446re (platform: ststm32; board: nucleo_f446re; framework: mbed)
...
省略
...
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

MicroPythonが起動すると、ボードのもうひとつのUSBにMicroPythonとの通信用の/dev/ttyACM0が現れる。
つまり、書き込みに使ったデバッガー用USBが電源供給用、もう一つのUSB(typeB)がMicroPythonの通信用になるので
２つのUSBをホストＰＣに接続する。接続後、USBストレージとして、PYBFLASHが出現する。ここに、boot.py,main.pyなどが
置かれる。

## 動作確認
picocomを使いボードとシリアルで通信する。
以下、通信例：
```bash

$ picocom /dev/ttyACM0 -b115200
#反応がない場合、Ctrl-Cを押す。
MicroPython v1.12-58-g7ef2f65-dirty on 2020-02-01; F4DISC with STM32F407
Type "help()" for more information.
>>> 

>>> import os
>>> os.uname()
(sysname='pyboard', nodename='pyboard', release='1.12.0', version='v1.12-58-g7ef2f65-dirty on 2020-02-01', machine='F4DISC with STM32F407')

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

>>> help("modules")
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

## サンプルスクリプト
blink.py
```python

print("LED blinking test...")
#led = pyb.LED(1) # Red LED on board
#led = pyb.LED(2) # Green LED on board
led = pyb.LED(3) # Orange LED on board
#led = pyb.LED(4) # Blue LED on board

while True:
    led.on()
    pyb.delay(200)
    led.off()
    pyb.delay(200)

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

本ボードの場合、USBストレージとしてPYBFLASHにファイルを読み書きできるので
テキストエディタでmain.pyや関連のファイルを作る。
(つまり、ampyをプログラム書き込みに使う必要はない)

picocomなどで接続中にPYBFLASHのファイルを編集できるので
テキストエディタでmain.pyを編集中に上書き保存し
picocomの画面を選択しCtrl-C(プログラム停止)、
Ctrl-D(soft-reboot)すると、編集中のmain.pyが
直ぐに実行される。これによりプログラムを修正しながら実行できるので
プログラミングの効率が向上する。

## 技術情報

[stm32f4-discovery-pinout](https://i0.wp.com/microcontrollerslab.com/wp-content/uploads/2019/12/stm32f4-discovery-pinout.png)   
[STM32F4 Discovery Board Pinout, Features and Examples](https://microcontrollerslab.com/stm32f4-discovery-board-pinout-features-examples/)   
[Quick reference for the pyboard](https://docs.micropython.org/en/latest/pyboard/quickref.html)   


LED/I2Cのpinout
```c++
// LEDs
#define MICROPY_HW_LED1 (pin_D14) // red
#define MICROPY_HW_LED2 (pin_D12) // green
#define MICROPY_HW_LED3 (pin_D13) // orange
#define MICROPY_HW_LED4 (pin_D15) // blue

/ I2C busses
#define MICROPY_HW_I2C1_SCL (pin_B6)
#define MICROPY_HW_I2C1_SDA (pin_B7)
#define MICROPY_HW_I2C2_SCL (pin_B10)
#define MICROPY_HW_I2C2_SDA (pin_B11)

```

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
