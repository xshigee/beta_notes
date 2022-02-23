
2020/2/18++++

BareMetal RpiZero MicroPython Install
# BareMetal RpiZero MicroPython Install

## 概要
まだWiFiがサポートされていないようだがBareMetalのMicropythonをRaspberryPi_Zeroにインストールする方法について記載する。
以下、RaspberryPi_ZeroをRpiZeroとする。
(ホストPCとしてはubuntuを想定している)

## 事前準備
(1)ampyのインストール   
AMPY_PORTは、自分の環境に合わせる。
```bash

pip install adafruit-ampy
export AMPY_PORT=/dev/ttyUSB0
export AMPY_BAUD=115200
```
(2)picocomのインストール
```bash

sudo apt-get install picocom
```
以上のうち、exportしているものは、.bashrcに登録することを勧める。


## ビルド手順
以下の手順でビルドする：
```bash

git clone https://github.com/boochow/micropython-raspberrypi.git
cd micropython-raspberrypi
git submodule update --init
cd micropython; git submodule update --init; cd ..
cd raspberrypi
make BOARD=RPI1

# Raspberry Pi2の場合、以下にする： 
#make BOARD=RPI2

# 以上で次のイメージが作成される
build/firmware.img

```

## ビルドしたファームウェアの書き込み
参照した手順ではブートできなかったので、実際にブートできたやり方を以下に記する：  
(1)本家サイトで記載されたやりかたでブートSDを作成する。  
[headless RaspberryPiインストール方法](https://beta-notes.way-nifty.com/blog/2020/01/post-2b8ed3.html)を参照のこと。   
(2)そのsdにはbootとrootfsのパーティションが作成されるが
bootに上でビルドしたfirmware.imgをコピーする。  
(3)bootにあるconfig.txtを編集して以下の１行を末尾に追加する。   
kernel=firmware.img  

参考情報：  
ネット情報によれば/boot のカーネルイメージには、以下がある：  
(1)kernel.img  
初代Raspberry Pi/RpiZeo用でarmv6になる  
(2)kernel7.img  
Raspberry Pi2用/Pi3用/Pi4?用でarmv7になる。  

　　
今回のブートは、kernel.imgの代わりに
firmware.imgを起動することになる。
　　
## BareMetal MicroPythonの起動
(1)シリアルをコンソールにするので、
以下のようにRpiZeroとUSBシリアルアダプターを接続する。
| RpiZero | USB_serial(AE-TTL-232R) |
| -------: | :--------- |
| p2(5V) |  NC |
| p4(5V) | NC |
| p6(GND) | GND |
| p8(TXD) | RXD |
| p10( RXD) | TXD |
NC:None Connection

接続は以下の写真が分かりやすい：  
[https://blog.boochow.com/wp-content/uploads/rpi-zero-serial.jpg](https://blog.boochow.com/wp-content/uploads/rpi-zero-serial.jpg)  

(2)USBシリアルをホストＰＣに接続する。(デバイス名を/dev/ttyUSB0とする)   
(3)「picocom /dev/ttyUSB0 -b115200」でUSBシリアルを起動しておく。  
(4)RpiZeroに上で作成したSDを刺し電源を入れる。(MicroPythonが起動する)  
(5)起動しておいたUSBシリアルにMicroPythonのメッセージが出力される。

## BareMetal MicroPython　ログ出力例
picocomを使いボードとシリアルで通信する。
以下、通信例：
```bash

PYB: soft reboot

mounting SD card...done
MicroPython 0fb6bf7 on 2020-02-18; Raspberry Pi with ARM1176JZF-S
Type "help()" for more information.

>>> import gc
>>> gc.collect()
>>> gc.mem_free()
65567440
# 他のボードよりもRAMが大きいので、大きな数字になっている。
>>> 

>>> 
>>> import os
>>> os.listdir()
['overlays', 'bcm2708-rpi-b-plus.dtb', 'COPYING.linux', 'LICENCE.broadcom', 'bcm2708-rpi-zero-w.dtb', 'bcm2708-rpi-b.dtb', 'bcm2708-rpi-cm.dtb', 'issue.txt', 'bcm2708-rpi-zero.dtb', 'bcm2709-rpi-2-b.dtb', 'bcm2710-rpi-2-b.dtb', 'bcm2710-rpi-3-b-plus.dtb', 'bcm2710-rpi-3-b.dtb', 'bcm2710-rpi-cm3.dtb', 'bcm2711-rpi-4-b.dtb', 'bootcode.bin', 'cmdline.txt', 'config.txt', 'fixup.dat', 'fixup4.dat', 'fixup4cd.dat', 'fixup4db.dat', 'fixup4x.dat', 'fixup_cd.dat', 'fixup_db.dat', 'fixup_x.dat', 'kernel.img', 'kernel7.img', 'kernel7l.img', 'kernel8.img', 'start.elf', 'start4.elf', 'start4cd.elf', 'start4db.elf', 'start4x.elf', 'start_cd.elf', 'start_db.elf', 'start_x.elf']
# 通常のブートsdのbootが見えているので、必要ないファイルが見えている。

>>> import sys
>>> sys.path
['', '/sd/lib', '/sd']
# パスはSD向けに設定されている
>>> 


>>> import os
>>> os.uname()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: no such attribute
# エラーになるようだ

>>> 
>>> list(5 * x + y for x in range(10) for y in [4, 2, 1])
[4, 2, 1, 9, 7, 6, 14, 12, 11, 19, 17, 16, 24, 22, 21, 29, 27, 26, 34, 32, 31, 39, 37, 36, 44, 42, 41, 49, 47, 46]
>>> 

>>> help('modules')
__main__          hashlib           struct            ujson
array             io                sys               uos
binascii          json              time              urandom
builtins          machine           ubinascii         ure
cmath             math              ucollections      ustruct
collections       mcu               uctypes           utime
errno             micropython       uerrno            utimeq
framebuf          os                uhashlib          uzlib
gc                random            uheapq            zlib
gpu               re                uio
Plus any modules on the filesystem
>>> 


```



## ampy実行例
以下のようにampyも動作するようだ：
```bash

$ ampy ls
/sd

~$ ampy ls /sd
/sd/COPYING.linux
...
省略
...
/sd/kernel.img

```

## 動作したスクリプト

LEDonBoard_blink.py
```python

import utime
from machine import Pin
# set gpio47 to output mode
led=Pin(47,Pin.OUT) # LED on board
while(1):
    led(0)  # LED on
    utime.sleep_ms(300)
    led(1)  # LED off
    utime.sleep_ms(300)
```
ボード上のLEDが点滅する。


performaceTestRpiZero.py
```python

# Peformace Test RpiZero
import time
def performanceTest():
    msec = time.ticks_ms
    endTime = msec() + 10000
    count = 0
    while msec() < endTime:
        count += 1
    print("Count: ", count)

performanceTest()
```
以下が出力される。(数字は多少変化する)  
Count:  680525  
  
Nucleo-F446REの数字が、2825625くらいなので
それに比べると随分遅い結果になる。  
(CPUキャッシュが有効になっていないせいだと思われる)  

https://github.com/micropython/micropython-unicorn/blob/master/www-emu/pyboard_demos.py
からの抜粋：  
mandel.py
```python

# A python Mandelbrot set courtesy of
# http://warp.povusers.org/MandScripts/python.html
# Try your own Python3 scripts on MicroPython!

minX = -2.0
maxX = 1.0
width = 60
height = 28
aspectRatio = 2

chars = ' .,-:;i+hHM$*#@ '

yScale = (maxX-minX)*(float(height)/width)*aspectRatio

for y in range(height):
    line = ''
    for x in range(width):
        c = complex(minX+x*(maxX-minX)/width, y*yScale/height-yScale/2)
        z = c
        for char in chars:
            if abs(z) > 2:
                break
            z = z*z+c
        line += char
    print(line)
```

## 動作しなかったスクリプト(エラーが出る)

led_blink.py
```python

import mcu, utime
# set gpio47 to output mode
mcu.mem32[mcu.GPFSEL4] = 2**21
while(1):
    mcu.mem32[mcu.GPCLR1] = 2**15
    utime.sleep_ms(300)
    mcu.mem32[mcu.GPSET1] = 2**15
    utime.sleep_ms(300)
```
mcu.mem32のところでエラーになる。

assembler.py
```python

# inline assembler
@micropython.asm_thumb
def asm_add(r0, r1):
    add(r0, r0, r1)

print(asm_add(1, 2))
```
inline-assemblerはサポートされていない？ようだ。

## 備考
ブートが安定しなくて、起動しないことが多い。原因は不明。一晩おいてから電源をいれるとブートしたりする。なにかしらリセット周りの問題？  
→問題解決した。(2020/2/19)   
「ブート問題解決(追記)」を参照のこと。   
  
完成度が高まり、WiFiなどが使えるようになれば、機能的にはESP32の最上位機種的なものとして使える印象がある。

## ブート問題解決(追記)
通常のブートSDをそのまま流用していたので、ブート時のMicroPythonのSDのマウントにおいて、rootfsのパーティションがあるので
マウントに失敗してブートしないと推測する。（そうすると、成功率が低いにせよ、マウントできてブートが正常になる理由が分からないという話になるが。。。）  
そこでrootfsがないSD(普通のフォーマットのSD)に以下のファイルのみをコピーして、それをブートSDとする。  

config.txt, firmware.img, start.elf, bootcode.bin  
補足：start.elf, bootcode.bin は、オリジナルの/bootに含まれたもの

このブートSDを使用することで確実にブートするようになった。  
以下、ブート時のREPL出力：
```bash

mounting SD card...done
MicroPython 0fb6bf7 on 2020-02-18; Raspberry Pi with ARM1176JZF-S
Type "help()" for more information.

>>> import os
>>> os.listdir()
['config.txt', 'firmware.img', 'start.elf', 'bootcode.bin']

>>> import gc
>>> gc.collect()
>>> gc.mem_free()
65565504
>>> 

>>> import sys
>>> sys.path
['', '/sd/lib', '/sd']
>>> 

```

なお、同じブートSDで初代Raspberry-PiでもMicroPythonが起動した。

## 参照情報

[MicroPython on bare metal Raspberry Pi Zero / Zero W / 2](https://github.com/boochow/micropython-raspberrypi)   
[PiZero pinout](https://cdn.sparkfun.com/assets/learn_tutorials/6/7/6/PiZero_1.pdf)  
[AE-TTL-232R-PINSOCKET](http://akizukidenshi.com/download/ds/akizuki/AE-TTL-232R-PINSOCKET.pdf)  
  
[Quick reference for the Raspberry Pi Zero W MicroPython](https://github.com/boochow/micropython-raspberrypi/wiki/Quick-reference-for-the-Raspberry-Pi-Zero---W-MicroPython)   
  
  
[ベアメタルRaspberry Pi用MicroPythonが公開](https://blog.boochow.com/article/455828824.html)  
[ベアメタルRaspberry Pi用MicroPythonをv1.11ベースに](https://blog.boochow.com/article/post-4237.html)  

[USB版](https://github.com/boochow/micropython-raspberrypi/releases/download/v1.0.0/20181124-rpi-usb.zip)  
HDMIディスプレイとUSBキーボードが使えるUSB版:  
試していないが以下で使用できるようだ。  
(1)microSDカードへ展開  
(2)main.pyを開き、コメントアウトを外し（’# ‘を削除）て保存  
(3)HDMIディスプレイとUSBキーボード（US配列のみ）を接続して起動  
  

以上
