
2020/4/11+  

STM32F4DISCOVERY CircuitPython Install
# STM32F4DISCOVERY CircuitPython Install

## 概要
STM32F4DISCOVERYにCircuitPythonをインストールする方法について記載する。
(ホストPCとしてはubuntuを想定している)

## 事前準備
(1)書き込みツール(st-link)のインストール
```bash

sudo apt install cmake
sudo apt install libusb-1.0

git clone https://github.com/texane/stlink.git
cd stlink
make
cd build/Release
sudo make install
sudo ldconfig

# インストール確認
st-flash --version
#以下のように出力されればＯＫ(数字などはバージョンによって異なる)
v1.6.0-154-g99a8aaa

```

(2)ampyのインストール   
AMPY_PORTは、自分の環境に合わせる。
```bash

pip install adafruit-ampy
export AMPY_PORT=/dev/ttyACM0
export AMPY_BAUD=115200
```

(3)picocomのインストール
```bash

sudo apt-get install picocom
```

(4)コンパイラのバージョンアップ   
以下でインストールしたコンパイラが古いのでバージョンアップする。  
sudo apt-get install gcc-arm-none-eabi

```bash
(1)以下のurlから最新版をダウンロードする：  
https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads

解凍したものを以下のフォルダに置く：   
$HOME/Downloads/gcc-arm-none-eabi-8-2019-q3-update

(2)パス設定  
#古いコンパイラを削除する  
sudo apt-get remove gcc-arm-none-eabi  
#パスを設定する  
export PATH=$PATH:$HOME/Downloads/gcc-arm-none-eabi-8-2019-q3-update/bin

```
以上のうち、exportしているものは、.bashrcに登録することを勧める。

## ビルド手順
以下の手順で行なう：
```bash

mkdir mp_cpy
cd mp_cpy

git clone https://github.com/adafruit/circuitpython.git
cd circuitpython
git submodule sync
git submodule update --init
git submodule update --recursive

make -C mpy-cross

cd ports/stm

make clean BOARD=stm32f4_discovery
make BOARD=stm32f4_discovery

```
以上の手順で、　./build-stm32f4_discovery/firmware.bin　が生成される 

stm32f411ve_discoveryの場合：
```bash

make clean BOARD=stm32f411ve_discovery
make BOARD=stm32f411ve_discovery


```

stm32f412zg_discoveryの場合：
```bash

make clean BOARD=stm32f412zg_discovery
make BOARD=stm32f412zg_discovery

```

## 書き込み手順
```bash

st-flash write ./build-stm32f4_discovery/firmware.bin 0x8000000

#出力例：
st-flash 1.6.0-154-g99a8aaa
2020-04-11T09:27:18 INFO common.c: F4xx: 192 KiB SRAM, 1024 KiB flash in 16 KiB pages.
2020-04-11T09:27:18 INFO common.c: Attempting to write 365636 (0x59444) bytes to stm32 address: 134217728 (0x8000000)
Flash page at addr: 0x08040000 erasedEraseFlash - Sector:0x6 Size:0x20000 
2020-04-11T09:27:24 INFO common.c: Finished erasing 7 pages of 131072 (0x20000) bytes
2020-04-11T09:27:24 INFO common.c: Starting Flash write for F2/F4/L4
2020-04-11T09:27:24 INFO flash_loader.c: Successfully loaded flash loader in sram
enabling 32-bit flash writes
size: 32768
size: 32768
size: 32768
size: 32768
size: 32768
size: 32768
size: 32768
size: 32768
size: 32768
size: 32768
size: 32768
size: 5188
2020-04-11T09:27:34 INFO common.c: Starting verification of write complete
2020-04-11T09:27:41 INFO common.c: Flash written and verified! jolly good!

```

書き込んだプログラムが実行されると、USBストレージとして、CIRCUITPYが現れる。  
注意：USBケーブルは、電源供給のためのデバッグボード側のUSB、CPU本体側のUSBの２つをホストPCに接続すること。

## 動作確認(REPL)
picocomを使いボードとシリアルで通信する。  

以下、通信例：
```bash

$ picocom /dev/ttyACM0 -b115200

Auto-reload is on. Simply save files over USB to run them or enter REPL to disable.

Press any key to enter the REPL. Use CTRL-D to reload.
Adafruit CircuitPython 5.0.0-87-g3a5f79a on 2020-04-10; STM32F4_DISCO with STM32F407VG
>>> 

>>> import os
>>> os.uname()
(sysname='stm32f4', nodename='stm32f4', release='5.0.0', version='5.0.0-87-g3a5f79a on 2020-04-10', machine='STM32F4_DISCO with STM32F407VG')
>>> 

>>> import gc
>>> gc.collect()
>>> gc.mem_free()
75392
>>> 

>>> list(5 * x + y for x in range(10) for y in [4, 2, 1])
[4, 2, 1, 9, 7, 6, 14, 12, 11, 19, 17, 16, 24, 22, 21, 29, 27, 26, 34, 32, 31, 39, 37, 36, 44, 42, 41, 49, 47, 46]
>>> 

>>> help('modules')
__main__          displayio         neopixel_write    terminalio
analogio          fontio            os                time
array             gc                pulseio           ulab
board             io                random            usb_hid
builtins          json              storage           usb_midi
busio             math              struct
collections       microcontroller   supervisor
digitalio         micropython       sys
Plus any modules on the filesystem
>>> 

>>> import board
>>> dir(board)
['__class__', 'A0', 'A1', 'A2', 'A3', 'A4', 'A5', 'LD3', 'LD4', 'LD5', 'LD6', 'LED1', 'LED2', 'LED3', 'LED4', 'LED_BLUE', 'LED_GREEN', 'LED_ORANGE', 'LED_RED', 'PA00', 'PA01', 'PA02', 'PA03', 'PA04', 'PA05', 'PA06', 'PA07', 'PA08', 'PA09', 'PA10', 'PA13', 'PA14', 'PA15', 'PB00', 'PB01', 'PB02', 'PB03', 'PB04', 'PB05', 'PB06', 'PB07', 'PB08', 'PB09', 'PB10', 'PB11', 'PB12', 'PB13', 'PB14', 'PB15', 'PC00', 'PC01', 'PC02', 'PC03', 'PC04', 'PC05', 'PC06', 'PC07', 'PC08', 'PC09', 'PC10', 'PC11', 'PC12', 'PC13', 'PC14', 'PC15', 'PD00', 'PD01', 'PD02', 'PD03', 'PD04', 'PD05', 'PD06', 'PD07', 'PD08', 'PD09', 'PD10', 'PD11', 'PD12', 'PD13', 'PD14', 'PD15', 'PE00', 'PE01', 'PE02', 'PE03', 'PE04', 'PE05', 'PE06', 'PE07', 'PE08', 'PE09', 'PE10', 'PE11', 'PE12', 'PE13', 'PE14', 'PE15']
>>> 


```

## スクリプト実行
blinkBlue.py
```python

import board
import digitalio
import time

led = digitalio.DigitalInOut(board.LED_BLUE)
led.direction = digitalio.Direction.OUTPUT

while True:
    led.value = True
    time.sleep(0.1)
    led.value = False
    time.sleep(0.1)

```
なお、LED_BLUEの部分は、LED_GREEN, LED_ORANGE, LED_REDに置き換えて
LEDの色を変えることができる。
  

ストレージのCIRCUITPYに上のblinkBlue.pyを書き込み
REPLで以下を実行すると該当のプログラムが実行される。   

```python

import blinkBlue
```
注意：実行前(import前)にControl-Dを押して、ボードを初期化する必要がある。  
(そうでないと、リソースが既に使用中だったりするとエラーになる)

CIRCUITPY上のcode.pyは、電源オン時またはREPLのCtrl-D押下時に、自動実行される。


## Adafruit_CircuitPython_Bundleのモジュールのインストール方法
```bash

# ホストＰＣ上(CircuitPythonボードが接続されている前提)
# パス設定
export CPY_LIB=$HOME/.local/lib/python3.5/site-packages
# 上の「python3.5」の部分は、自分の使用している(ホストPCの)python3のバージョンに合わせる
export CPY=/media/$USER/CIRCUITPY

# MODULEというモジュールをインストールする場合、以下を実行する：
pip3 install adafruit-circuitpython-MODULE
# CircuitPython上でimportする場合のスクリプト
import adafruit_MODULE
# ダッシュ(-)とアンダースコア(_)の使い分けに注意すること

# モジュールのCircuitPythonボードへのコピー(インストール)
cp $CPY_LIB/adafruit_MODULE.py $CPY/lib/

# 具体例：
pip3 install adafruit-circuitpython-mpl115a2
cp $CPY_LIB/adafruit_mpl115a2.py $CPY/lib/

```


## Pin Assignment

I2C pin:

```

PB06 = I2C1_SCL
PB09 = I2C1_SDA

PB10 = I2C2_SCL
PB11 = I2C2_SDA

PA08 = I2C3_SCL
PC09 = I2C3_SDA

```

sample code for i2c
```python
import time
import busio
import board

i2c = busio.I2C(board.PB06, board.PB09) # I2C1 for F4DISCO
#i2c = busio.I2C(board.PB10, board.PB11) # I2C2 for F4DISCO
#i2c = busio.I2C(board.PA08, board.PC09) # I2C3 for F4DISCO

i2c.try_lock()

i2c.scan()

```

SPI pin:
```

PA04 = SPI1_NSS
PA05 = SPI1_SCK
PA07 = SPI1_MOSI
PA06 = SPI1_MISO

PB12 = SPI2_NSS
PB13 = SPI2_SCK
PB15 = SPI2_MOSI
PB14 = SPI2_MISO

```

SPI設定例：

```
spi = busio.SPI(board.PA05, MOSI=board.PA07, MISO=board.PA06) # SPI1
spi = busio.SPI(board.PB13, MOSI=board.PB15, MISO=board.PB14) # SPI2
```


AnalogIn pin:
```

PC01 = ADC123_IN11
PC02 = ADC123_IN12
PC05 = ADC12_IN15
PC04 = ADC12_IN14
PB01 = ADC12_IN9
PB00 = ADC12_IN8

```

Sample Code for AnalogIn
```python
import time
import board
from analogio import AnalogIn

analog_in = AnalogIn(board.PC01)

def get_voltage(pin):
    return (pin.value * 3.3) / 65536

while True:
    print(get_voltage(analog_in))
    time.sleep(0.1)
```


GPIO pin:
```

PE07 = FSMC_D4
PE08 = FSMC_D5
PE09 = FSMC_D6
PE10 = FSMC_D7
PE11 = FSMC_D8
PE12 = FSMC_D9
PE13 = FSMC_D10
PE14 = FSMC_D11
PE15 = FSMC_D12
PD08 = FSMC_D13
PD09 = FSMC_D14
PD10 = FSMC_D15
PD11 = FSMC_A16
PD12 = FSMC_A17
PD13 = FSMC_A18
PD14 = FSMC_D0
PD15 = FSMC_D1

LED Green  = PD12
LED Orange = PD13
LED Red    = PD14
LED Blue   = PD15

User Button = PA00 # User and Wake-Up button
# 使用する際は、プルアップしない。押すとTrueになる。
# LEDは負論理なのでボタンに押すとLEDが消える。
# 他のピンでブルアップしたいる場合は、
# そのピンをグランドに落とすとＬＥＤが光る。

```

Sample Code for GPIO
```python

import time
import board
from digitalio import DigitalInOut, Direction, Pull

led = DigitalInOut(board.PD13)
led.direction = Direction.OUTPUT

switch = DigitalInOut(board.PA00)
switch.direction = Direction.INPUT
#switch.pull = Pull.UP   # Pull.Down is available on some MCUs

while True:
    led.value = not switch.value
    print(switch.value)
    time.sleep(0.01)

```

## 参照URL

[download STM32-Discovery-F4 PINOUT(.xls)](http://busboard.us/products/PCB-STM32-F4B1/Kornak-(STM32-Discovery-F4)-0001%20Rev%201.01%20Module%20Pinouts%20&%20Functions.xls)   

[ＳＴＭ３２Ｆ４ＤＩＳＣＯＶＥＲＹ](http://akizukidenshi.com/catalog/g/gM-05313/)   

[Discovery kit with STM32F407VG MCU](https://www.st.com/resource/en/data_brief/stm32f4discovery.pdf)  
[Discovery kit with STM32F411VE MCU](https://www.st.com/resource/en/user_manual/dm00148985-discovery-kit-with-stm32f411ve-mcu-stmicroelectronics.pdf)  
[Discovery kit with STM32F412ZG MCU](https://www.st.com/content/ccc/resource/technical/document/user_manual/group0/93/30/cb/96/f7/1c/4c/d2/DM00275919/files/DM00275919.pdf/jcr:content/translations/en.DM00275919.pdf)  

[Build CircuitPython](https://learn.adafruit.com/building-circuitpython/build-circuitpython)  
[Adding Frozen Modules](https://learn.adafruit.com/building-circuitpython/adding-frozen-modules)  

[Adafruit CircuitPython API Reference(v4.x)](https://circuitpython.readthedocs.io/en/4.x/docs/index.html)  

[CircuitPython Essentials](https://learn.adafruit.com/circuitpython-essentials)  
[Example Code](https://github.com/adafruit/Adafruit_Learning_System_Guides/tree/master/CircuitPython_Essentials)  


[CircuitPython Cheatsheet](https://github.com/adafruit/awesome-circuitpython/blob/master/cheatsheet/CircuitPython_Cheatsheet.md)   


以上
