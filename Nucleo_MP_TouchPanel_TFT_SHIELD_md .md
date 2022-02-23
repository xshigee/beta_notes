
2020/2/29

Nucleo MicroPython TouchPanel of TFT Shield
# Nucleo MicroPython TouchPanel of TFT Shield

## 概要
NucleoのMicroPythonに以下のTFT_Shieldを接続して、タッチパネルを使う。
ここでは、linux環境でのインストール方法について説明する。   

[2.8 inch TFT Touch Shield v2.0](http://wiki.seeedstudio.com/2.8inch_TFT_Touch_Shield_v2.0/)   
(LCDコントローラ:ILI9341)  

## 配線表
実際にはシールドを刺すだけだが以下のように配線したことになる。

| Nucleo/Arduino_Pin | Function | 
| :--: | :--: |
| D4 | TF_CS/SD_CS |
| D5 | TFT_CS |
| D6 | TFT_DC |
| D7 | BACKLIGHT(Selectable)(NOT USED) |
| D10 | RST(dummy)|
| D11 | MOSI |
| D12 | MISO |
| D13 |	SCK |

| Nucleo/Arduino_Pin | Function | 
| :--: | :--: |
| A0 | TOUCH PANEL(YM/Y-) |
| A1 | TOUCH PANEL(XM/X-） |
| A2 | TOUCH PANEL(YP/Y+) |
| A3 | TOUCH PANEL(XP/X+) |


## rshellのインストール
ampyでフラッシュに書き込めない状況があったので、その対策として、rshellをインストールして、それを使用してフラッシュにスクリプトを書き込む。(ampyで書き込めない時にrshellで書き込めた場合があったが、ampyで特に問題ない場合、使用する必要はない)  
以下を実行してインストールする：
```bash

sudo pip3 install rshell
export RSHELL_PORT=/dev/ttyACM0

```
使い方については以下のリンクを参照のこと：  
[Remote MicroPython shell.](https://github.com/dhylands/rshell)   


## タッチパネルの座標を取得するスクリプト

tp_test5.py
```python

# tp_test5.py
# display xy pos
# followings are measured values by ADC
xminTP = 192
yminTP = 144
xmaxTP = 22229
ymaxTP = 33000
#
xmin = 10000
ymin = 10000
xmax = 0
ymax = 0
from math import floor
from time import sleep
from machine import Pin, ADC
while True:
   ym = Pin('A0', Pin.OUT_PP)
   xp = Pin('A3', Pin.OUT_PP)
   xm=ADC('A1')
   #
   xp.value(0) # Set X+ to ground
   ym.value(1) # Set Y- to VCC
   yp= Pin('A2', Pin.OUT, Pin.PULL_NONE) # want Hi-Z but ...
   yp.value(1)
   xraw=xm.read_u16()
   #print('xraw:'+str(xraw)) # debug
   if xraw <= 128:
      xpos = -1
   else:
      xmax = xraw if xraw>xmax else xmax
      xmin = xraw if xraw<xmin else xmin
      xpos = floor(abs((xraw-xminTP)/(xmaxTP-xminTP)*240))
      xpos = 240 - xpos
      xpos = 0 if xpos<0 else xpos
   print('x:'+str(xpos))
   #print('xmin:'+str(xmin)) # debug
   #print('xmax:'+str(xmax)) # debug
   #
   ym = Pin('A0', Pin.OUT_PP)
   xp = Pin('A3', Pin.OUT_PP)
   xm=Pin('A1', Pin.OUT_PP)
   yp=ADC('A2')
   #
   ym.value(0) # Set Y- to ground
   xm.value(1) # Set X+ to VCC
   xp= Pin('A3', Pin.OUT, Pin.PULL_NONE) # want Hi-Z but ...
   xp.value(1)
   yraw=yp.read_u16()
   #print('yraw:'+str(yraw)) # debug
   if yraw <= 128:
      ypos = -1
   else:
      ymax = yraw if yraw>ymax else ymax
      ymin = yraw if yraw<ymin else ymin
      ypos = floor(abs((yraw-yminTP)/(ymaxTP-yminTP)*320))
   print('Y:'+str(ypos))
   #print('ymin:'+str(ymin)) # debug
   #print('ymax:'+str(ymax)) # debug
   print('--------------------')
   sleep(0.1)


```
触っていない時は、-1を座標値と返す。(x,y)で(0,0)から(239, 319)を返すのが理想だが、まだ、調整不足なので、そうなっていない。   
なお、USBコネクタ側を下にして縦長(ポートレート)に持ち、左上が原点になる。

## 実行

```bash

# rshellの場合
$ rshell
Using buffer-size of 32
Connecting to /dev/ttyACM0 (buffer-size 32)...
Trying to connect to REPL  connected
Testing if ubinascii.unhexlify exists ... Y
Retrieving root directories ... /flash/
Setting time ... Feb 28, 2020 22:56:09
Evaluating board_name ... pyboard
Retrieving time epoch ... Jan 01, 2000
Welcome to rshell. Use Control-D (or the exit command) to exit rshell.

# スクリプトの書き込み
cp tp_test5.py /flash/tp_test5.py

# REPLに入る
repl
Entering REPL. Use Control-X to exit.
>
MicroPython v1.12-58-g7ef2f65-dirty on 2020-01-30; NUCLEO-F446RE with STM32F446xx
Type "help()" for more information.
>>> 
# 実行 
>>> import tp_test5

# REPLから抜けるときは、Ctrl-Xを入力する
# さらにrshellから抜けるときはCtrl-Dを入力する

```

SDから実行する場合は、REPLで以下を実行する：  
(sdcard.pyがフラッシュに置いてある前提)
```python

# SD setup (Nucleo)
from machine import Pin, SPI
from sdcard import SDCard
import os, sys
spi = SPI(miso=Pin('D12'), mosi=Pin('D11', Pin.OUT), sck=Pin('D13', Pin.OUT))
sd = SDCard(spi, Pin('D4'))
os.mount(sd, '/sd')
sys.path.append('/sd')
sys.path.append('/sd/lib')
os.chdir('/sd')
#os.listdir()
import tp_test5

```


## フラッシュ書き込み問題の解決方法
Nucleoだけの問題の可能性があるが、ときどきフラッシュの容量があるにもかかわらず、容量が足りないことによるエラーがでて書き込みが正常終了しないことがあった。この際は、いったんフラッシュの内容を全クリアしてから書き込むと正常に行くようだ。  
rshellを使用しているならば、以下のように行なう：
```bash
$ rshell
Using buffer-size of 32
Connecting to /dev/ttyACM0 (buffer-size 32)...
Trying to connect to REPL  connected
Testing if ubinascii.unhexlify exists ... Y
Retrieving root directories ... /flash/
Setting time ... Feb 28, 2020 22:56:09
Evaluating board_name ... pyboard
Retrieving time epoch ... Jan 01, 2000
Welcome to rshell. Use Control-D (or the exit command) to exit rshell.

# フラッシュの内容を確認する
ls /flash

# フラッシュの内容を消去する
rm /flash/*.*

# 必要なスクリプトを書き込む
cp xxxx.py /flash/xxxx.py


```

## 参考情報  

[NucleoのMicroPythonにTFT_Shieldを接続する](https://beta-notes.way-nifty.com/blog/2020/02/post-f7b8d8.html)  

[Nucleo-L476RGにMicroPythonをインストールする](https://beta-notes.way-nifty.com/blog/2020/02/post-5995da.html)   
[Nucleo-F401REにMicroPythonをインストールする](https://beta-notes.way-nifty.com/blog/2020/02/post-8bfd28.html)  
[STM32F4-Discovery」にMicroPythonをインストールする](https://beta-notes.way-nifty.com/blog/2020/02/post-6822aa.html)  
[NUCLEO F446RE MicroPython インストール方法](https://beta-notes.way-nifty.com/blog/2020/01/post-459022.html)  
[パッチ情報 - NUCLEO F446RE MicroPython I2C接続確認](https://beta-notes.way-nifty.com/blog/2020/02/post-4dc66d.html)   


[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)　　　

[2.8 inch TFT Touch Shield v2.0](http://wiki.seeedstudio.com/2.8inch_TFT_Touch_Shield_v2.0/)   

他のILI9341サポートモジュール：   
[Micropython Driver for ILI9341 display](https://github.com/jeffmer/micropython-ili9341.git)  
[Micropython TFT Display Driver for ILI9341 Chipset](https://github.com/tkurbad/micropython-ili9341.git)  
Arduino用ライブラリー：  
[Seeed-Studio/TFT_Touch_Shield_V2/archive/master.zip](https://github.com/Seeed-Studio/TFT_Touch_Shield_V2/archive/master.zip)   


以上

