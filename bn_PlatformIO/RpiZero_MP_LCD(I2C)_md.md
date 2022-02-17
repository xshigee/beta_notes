
2020/2/21

RpiZero-MicroPython LCD(I2C) demo  
# RpiZero-MicroPython LCD(I2C) demo

## 概要
RpiZeroにGroveコネクタ経由で「Grove - LCD RGB Backlight」(I2C)を接続し、
BareMetal-MicroPythonで文字を表示する。

## 参照URL

[BareMetalのMicropythonをRaspberryPi_Zeroにインストールしてみる](https://beta-notes.way-nifty.com/blog/2020/02/post-573471.html)   
[Grove - LCD RGB Backlight](https://www.seeedstudio.com/Grove-LCD-RGB-Backlight.html)   


## 関連モジュールのインストール
以下のようにソースを入手しパッチを当ててからモジュールとしてインストールする。

```bash
git clone https://github.com/Bucknalla/micropython-i2c-lcd.git
cd micropython-i2c-lcd/lib

ls
i2c_lcd.py  i2c_lcd_screen.py ...
# 上の２つのファイルにパッチを当てる
i2c_lcd.py:
14行目付近の「i2c=...」をコメントアウトする。「#  i2c = I2C(0, I2C.MASTER)」
i2c_lcd_screen.py:
53行目付近の「i2c.init(...」をコメントアウトする。「#  i2c.init(I2C.MASTER, baudrate=20000)」

#以下の３つのファイルをBareMetal-MicroPythonのブートSDにコピーする
i2c_lcd.py　　
i2c_lcd_backlight.pｙ　　
i2c_lcd_screen.py　　

```

## デモ・スクリプト
RpiZeroボードに「Grove - LCD RGB Backlight」を以下の接続表のように接続し、LCDに文字を表示する。

I2Cの接続表

| pin | Function | Grove color |
| :---: | :----: | :---: |
| p1/p2 | 3.3V/5V  | RED |
| p3 | SDA(GP2)  | WHITE |
| p5 | SCL(GP3)  | YELLOW |
| p9 | GND   | BLACK |

今回のLCDの場合、5V電源でないと動作しないので
p2(5V)に接続する。

RpiZeroのピン配列は以下を参照のこと：  
[Quick reference for the Raspberry Pi Zero W MicroPython](https://github.com/boochow/micropython-raspberrypi/wiki/Quick-reference-for-the-Raspberry-Pi-Zero---W-MicroPython)  


以下のファイル(piZero_demo_i2c_LCD.py)も   
BareMetal-MicroPythonのブートSDにコピーする。

RpiZero_demo_i2c_LCD.py
```python

# Note: VCC needs 5V
import i2c_lcd
from machine import Pin,I2C
from time import sleep

i2c = I2C(1)

d = i2c_lcd.Display(i2c)

d.clear()

d.home()
d.write('LCD I2C test .py')
d.move(0,1)
d.write('Hello World        ')
d.color(0xFF,0,0)
sleep(0.5)
d.color(0,0xFF,0)
sleep(0.5)
d.color(0,0,0xFF)
sleep(0.5)
d.color(0x44,0xFF,0x44)
```


## 実行
RpiZeroボードに上で作ったブートSDを刺して電源を入れる。  
(いうまでもないが、USBシリアルとI2C接続が完了している前提)


以下のように起動後  
「import RpiZero_demo_i2c_LCD」を  
入力してプログラムを実行する。  
(LCDに文字が表示される)
```bash

mounting SD card...done
MicroPython 0fb6bf7 on 2020-02-18; Raspberry Pi with ARM1176JZF-S
Type "help()" for more information.

>>> import RpiZero_demo_i2c_LCD
>>> 


```


以上
