


2020/1/22

MicroPython Module Install 
# MicroPython Module Install 

## 概要
MicroPythonの外部モジュールのインストール方法について記する。microbitのMicroPythonと他のMicroPythonで方法が異なるので、２つに分けて説明する。
ここでは、linux環境でのインストール方法について説明する。   

## 準備
(1)microbit-MicroPython用ツール(ufs)のインストール
```bash

pip install microfs
pip install --no-cache --upgrade micro
```

(2)他のMicroPython用ツール(ampy)のインストール   
AMPY_PORTは、自分の環境に合わせる。
```bash

pip install adafruit-ampy
export AMPY_PORT=/dev/ttyUSB0
export AMPY_BAUD=115200
```

(3)piccomのインストール
MicroPyhonのコンソールとして使用する。
```bash

sudo apt-get install piccom
```
以上のうち、exportしているものは、.bashrcに登録することを勧める。

## microbit MicroPythonの場合
xxxx.pyのモジュールがあった場合、
microbitを接続して以下のコマンドを実行する。
```bash

# モジュールの書き込み
ufs put xxxx.py

# microbit内のファイルを確認する
ufs ls

# xxxx.pyのモジュールを使っているmmm.pyの書き込み
# (mmm.pyをmain.pyとして書き込む)
ufs mmm.py main.py

```

## 他のMicroPythonの場合
xxxx.pyのモジュールがあった場合、
デバイスを接続して以下のコマンドを実行する。
```bash

# モジュールの書き込み
ampy put /flash/xxxx.py

# デバイス内のファイルを確認する
ampy ls /flash

# xxxx.pyのモジュールを使っているmmm.pyの実行(RAM内)
ampy run mmm.py

# xxxx.pyのモジュールを使っているmmm.pyの書き込み
# (mmm.pyをmain.pyとして書き込む)
ampy put mmm.py /flash/main.py

```

## 実例＃１
microbit+Grobe_Shield_for_microbitに
Grove-Bar-LEDをslot「P2/P16」に刺して
Grove-Bar-LEDを動かす。

```bash

git clone https://github.com/mcauser/microbit-my9221.git

cd microbit-my9221/

# モジュールのインストール
ufs put my9221.py

ufs ls
my9221.py ...

# プログラムの書き込み
ufs put counter.py main.py
# 書き込み完了後、[reset]を押しプログラム起動する

```

接続に合わせて変更したcounter.py
```python
 
from microbit import *
from my9221 import MY9221

# slot[P2/P16]にGrove-Bar-LEDを接続する
ledbar = MY9221(di=pin2, dcki=pin16, reverse=False)

num = 0
ledbar.level(num)

while True:
    if button_a.was_pressed():
        num = num + 1
        ledbar.level(num)
    elif button_b.was_pressed():
        num = num - 1
        ledbar.level(num)

```

## 実例＃２
Nucleo_F446RE+Grove-Base_ShieldにGrove-Bar-LEDをslot#5に刺して
Grove-Bar-LEDを動かす。

```bash

git clone https://github.com/mcauser/micropython-my9221.git

cd micropython-my9221/

# モジュールをインストールする
ampy put my9221.py

ampy ls /flash
/flash/boot.py
...
/flash/my9221.py

# RAM実行
ampy run mp_my9221_test.py

# プログラム書き込み
mpy put mp_my9221_test.py　/flash/main.py

```

my9221_test.pyは
Nucle_F446RE+Grove-Base_Shieldに
合わせて以下のように変更する：
(mp_my9221_test.pyにrename)

mp_my9221_test.py
```python

from machine import Pin
from my9221 import MY9221

# ESP8266
# ledbar = MY9221(Pin(4), Pin(5))

# PyBoard
# ledbar = MY9221(Pin('X8'), Pin('X6'))

# STM32F446RE + Grove-Base Shield
ledbar = MY9221(Pin('B4'), Pin('B10')) # Digital I/O slot#5(D5,D6)

# all LEDS on, full brightness
ledbar.level(10)

# four LEDS on, half brightness
ledbar.level(4, 0x0F)

# reverse orientation, first LED is green
ledbar.reverse(True)
ledbar.level(1)

# normal orientation, first LED is red
ledbar.reverse(False)
ledbar.level(1)

# switch on specific leds
ledbar.bits(0b1111100000)
ledbar.bits(0b0000011111)
ledbar.bits(1)
ledbar.bits(3)
ledbar.bits(7)

# first and last LED on, very dim
ledbar.bits(513, 7)

# alternating LEDs
ledbar.bits(0b0101010101)
ledbar.bits(0b1010101010)
buf = b'\x00\xff\x00\xff\x00\xff\x00\xff\x00\xff'
ledbar.bytes(buf)

# fade out LEDs
buf = bytearray([0,1,3,7,15,31,63,127,255,255])
ledbar.reverse(True)
ledbar.bytes(buf)
ledbar.reverse(False)
ledbar.bytes(buf)

# various brightnesses
buf = [0,0,0,0,0,255,127,63,15,7]
ledbar.bytes(buf)

# cycle through LEDS with various brightnesses
from time import sleep_ms
buf = [0,1,3,7,15,31,63,127,255,255]
for i in range(50):
    buf.insert(0,buf.pop())
    ledbar.bytes(buf)
    sleep_ms(100)

# random LEDs
import urandom
for i in range(100):
    ledbar.bits(urandom.getrandbits(10))

# walk through all possible LED combinations
for i in range(1024):
    ledbar.bits(i)

# Use 8bit greyscale mode (default)
# LED brightness 0x00-0xFF
ledbar._write16(0x00) # command
ledbar._write16(0xFF) # led 1
ledbar._write16(0xFF) # led 2
ledbar._write16(0x00) # led 3
ledbar._write16(0x00) # led 4
ledbar._write16(0x00) # led 5
ledbar._write16(0xFF) # led 6
ledbar._write16(0xFF) # led 7
ledbar._write16(0x00) # led 8
ledbar._write16(0x00) # led 9
ledbar._write16(0x00) # led 10
ledbar._write16(0x00) # unused channel, required
ledbar._write16(0x00) # unused channel, required
ledbar._latch()

# Use 12bit greyscale mode
# LED brightness 0x000-0xFFF
ledbar._write16(0x0100) # command
ledbar._write16(0x0FFF) # led 1
ledbar._write16(0x0000) # led 2
ledbar._write16(0x00FF) # led 3
ledbar._write16(0x0000) # led 4
ledbar._write16(0x000F) # led 5
ledbar._write16(0x000F) # led 6
ledbar._write16(0x0000) # led 7
ledbar._write16(0x00FF) # led 8
ledbar._write16(0x0000) # led 9
ledbar._write16(0x0FFF) # led 10
ledbar._write16(0x0000) # unused channel, required
ledbar._write16(0x0000) # unused channel, required
ledbar._latch()

# Use 14bit greyscale mode
# LED brightness 0x000-0x3FFF
ledbar._write16(0x0200) # command
ledbar._write16(0x3FFF) # led 1
ledbar._write16(0x03FF) # led 2
ledbar._write16(0x0000) # led 3
ledbar._write16(0x0000) # led 4
ledbar._write16(0x0000) # led 5
ledbar._write16(0x003F) # led 6
ledbar._write16(0x0003) # led 7
ledbar._write16(0x0000) # led 8
ledbar._write16(0x0000) # led 9
ledbar._write16(0x0000) # led 10
ledbar._write16(0x0000) # unused channel, required
ledbar._write16(0x0000) # unused channel, required
ledbar._latch()

# Use 16bit greyscale mode
# LED brightness 0x0000-0xFFFF
ledbar._write16(0x0300) # command
ledbar._write16(0xFFFF) # led 1
ledbar._write16(0x0FFF) # led 2
ledbar._write16(0x00FF) # led 3
ledbar._write16(0x000F) # led 4
ledbar._write16(0x0007) # led 5
ledbar._write16(0x0003) # led 6
ledbar._write16(0x0001) # led 7
ledbar._write16(0x0000) # led 8
ledbar._write16(0x0000) # led 9
ledbar._write16(0x0000) # led 10
ledbar._write16(0x0000) # unused channel, required
ledbar._write16(0x0000) # unused channel, required
ledbar._latch()

```

## 参考情報  

[Grove - LED Bar](http://wiki.seeedstudio.com/Grove-LED_Bar/)   
[micro:bit用GROVEシールド v2.0](https://www.switch-science.com/catalog/5434/)   
[Grove - Base Shield v1.2](https://seeeddoc.github.io/Grove-Base_Shield_v1.2/)  
[MicroFS](https://github.com/ntoll/microfs)  [ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)　　　

以上
