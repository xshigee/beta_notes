
2020/2/28

Nucleo MicroPython TFT Shield
# Nucleo MicroPython TFT Shield

## 概要
NucleoのMicroPythonに以下のTFT_Shieldを接続する。SDスロットを持っているのでSDもサポートする。Touchパネルもあるが
ここでは、サポートしない。
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
| A0 | TOUCH PANEL(NOT SUPPORTED) |
| A1 | TOUCH PANEL(NOT SUPPORTED) |
| A2 | TOUCH PANEL(NOT SUPPORTED) |
| A3 | TOUCH PANEL(NOT SUPPORTED) |

## rshellのインストール
ampyでフラッシュに書き込めない状況があったので、その対策として、rshellをインストールして、それを使用してフラッシュにスクリプトを書き込む。(ampyで書き込めない時にrshellで書き込めた場合があったが、ampyで特に問題ない場合、使用する必要はない)  
以下を実行してインストールする：
```bash

sudo pip3 install rshell
export RSHELL_PORT=/dev/ttyACM0

```
使い方については以下のリンクを参照のこと：  
[Remote MicroPython shell.](https://github.com/dhylands/rshell)   


## 関連モジュールのインストール
以下の手順でインストールする：
```bash
# TFTモジュールのインストール
hg clone https://bitbucket.org/thesheep/micropython-ili9341
cd micropython-ili9341
ampy put ili9341.py

# rshellの場合
cp ili9341.py /flash/ili9341.py

# SD card モジュールのインストール
wget https://raw.githubusercontent.com/micropython/micropython/master/drivers/sdcard/sdcard.py
ampy put sdcard.py

# rshellの場合
cp sdcard.py /flash/sdcard.py

```

## 動作確認用スクリプト

TFT_SD_test_Nucleo.py
```python
# TFT Touch Shield(with SD)
# (does not support Touch)
# setup display module
import ili9341
color565 = ili9341.color565
from machine import Pin, SPI
spi = SPI(miso=Pin('D12'), mosi=Pin('D11', Pin.OUT), sck=Pin('D13', Pin.OUT))
#spi=SPI(1)
display = ili9341.ILI9341(spi, cs=Pin('D5'), dc=Pin('D6'), rst=Pin('D10'))
# setup SDCard module
import machine, sdcard, os
sd = sdcard.SDCard(spi, Pin('D4'))
os.mount(sd, '/sd')
# setup path
import sys
sys.path.append('/sd')
sys.path.append('/sd/lib')
# display test
display.fill(color565(0xff, 0x11, 0x22))
display.fill(color565(0xff, 0xff, 0xff))
display.fill(color565(0x00, 0x00, 0x00))
display.pixel(120, 160, color565(0xff,0,0))
display.pixel(121, 161, color565(0,0,0xff))
display.pixel(123, 163, color565(0,0xff,0))
display.text('Hello World!',0,0,color=color565(0,0xff,00))
display.text('0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz{}()+*<>?_',100,200,color=color565(0,0,0xff))
display.fill_rectangle(100,150,20, 50, color565(0xff,0,0))
# SD Card test
display.fill(color565(0x00, 0x00, 0xff))
display.text(str(sys.path),0,0,color565(0xff,0xff,0xff),clear_eol=True)
display.text(str(os.listdir('/sd')),0,0,color565(0xff,0xff,0xff),clear_eol=True)

```

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
cp TFT_SD_test_Nucleo.py /flash/TFT_SD_test_Nucleo.py

# REPLに入る
repl
Entering REPL. Use Control-X to exit.
>
MicroPython v1.12-58-g7ef2f65-dirty on 2020-01-30; NUCLEO-F446RE with STM32F446xx
Type "help()" for more information.
>>> 
# 実行 
>>> import TFT_SD_test_Nucleo

# REPLから抜けるときは、Ctrl-Xを入力する
# さらにrshellから抜けるときはCtrl-Dを入力する

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
cp xxxx.py /flash/xxxx/py


```

## 参考情報  

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

