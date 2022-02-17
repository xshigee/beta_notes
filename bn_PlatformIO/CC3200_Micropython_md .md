
2020/2/14+

CC3200 MicroPython Install
# CC3200 MicroPython Install

## 概要
以下のリンクにある「WiFi CC3200 Lauchpad」ボード用にMicroPythonをビルドして、そのfirewareを書き込む方法について述べる。
ここでは、linux環境でのインストール方法について説明する。   

[SimpleLink Wi-Fi CC3200 LaunchPad](http://www.tij.co.jp/tool/jp/CC3200-LAUNCHXL)  

## 準備
環境整備として以下をインストールする：  
(1)書き込みツール(cc3200tool)のインストール   

```bash

pip install git+git://github.com/ALLTERCO/cc3200tool.git

```
使い方は以下を参照のこと：  
[CC3200 Tool/README](https://github.com/ALLTERCO/cc3200tool)  
　

(2)ampyのインストール   
AMPY_PORTは、自分の環境に合わせる。
```bash

pip install adafruit-ampy
export AMPY_PORT=/dev/ttyUSB0
export AMPY_BAUD=115200
```

(3)picocomのインストール
```bash

sudo apt-get install picocom
```
以上のうち、exportしているものは、.bashrcに登録することを勧める。

## firewareのビルド
```bash

mkdir mp
cd mp

git clone https://github.com/micropython/micropython.git

cd micropython
cd mpy-cross
make

cd ports/cc3200 
make BTARGET=bootloader BTYPE=release BOARD=LAUNCHXL
make BTARGET=application BTYPE=release BOARD=LAUNCHXL
# 以上で次の２つのbinが生成される
# bootmgr/build/LAUNCHXL/release/bootloader.bin
# build/LAUNCHXL/release/mcuimg.bin

```

## firmwareの書き込み
書き込む前にボードのジャンパーが以下になっていることを確認する：  
J6/J7:シリアルが有効(たぶん工場出荷時のまま)    
SOP2: 100  
以下の写真どおりになっていればＯＫ.  
[J6/J7/SOP2設定写真](https://processors.wiki.ti.com/images/4/48/CC3200-LAUNCHXL_SOP2.jpeg)  


以下を実行する：
```bash

cd micropython/ports/cc3200

# format the flash, upload two binaries
cc3200tool -p /dev/ttyUSB0 --sop2 ~rts --reset dtr \
    format_flash --size=1M \
    write_file bootmgr/build/LAUNCHXL/release/bootloader.bin /sys/mcuimg.bin \
    write_file build/LAUNCHXL/release/mcuimg.bin /sys/factimg.bin

#出力ログ：
>     format_flash --size=1M \
>     write_file bootmgr/build/LAUNCHXL/release/bootloader.bin /sys/mcuimg.bin \
>     write_file build/LAUNCHXL/release/mcuimg.bin /sys/factimg.bin
2020-02-13 21:15:25,487 -- Connecting to target...
2020-02-13 21:15:25,844 -- Connected, reading version...
2020-02-13 21:15:25,852 -- connected to target
2020-02-13 21:15:25,853 -- Version: CC3x00VersionInfo((0, 4, 1, 2), (0, 0, 0, 0), (0, 0, 0, 0), (0, 0, 0, 0), (16, 0, 0, 0))
2020-02-13 21:15:25,853 -- This is a CC3200 device
2020-02-13 21:15:25,853 -- Switching to NWP bootloader...
2020-02-13 21:15:25,869 -- Switching UART to APPS...
2020-02-13 21:15:25,884 -- Resetting communications ...
2020-02-13 21:15:27,147 -- Uploading rbtl3100s.dll...
2020-02-13 21:15:27,149 -- Getting storage list...
2020-02-13 21:15:27,163 -- Getting storage info...
2020-02-13 21:15:27,179 -- storage #0 info bytes: 0x10, 0x0, 0x0, 0x10, 0x0, 0x0, 0x0, 0x0
2020-02-13 21:15:27,898 -- APPS version: CC3x00VersionInfo((0, 4, 0, 2), (0, 0, 0, 0), (0, 0, 0, 0), (0, 0, 0, 0), (16, 0, 0, 0))
2020-02-13 21:15:27,898 -- Formatting flash with size=1024
2020-02-13 21:15:28,953 -- Uploading file bootmgr/build/LAUNCHXL/release/bootloader.bin -> /sys/mcuimg.bin [11672, disk=11672]...
...
2020-02-13 21:15:29,417 -- Uploading file build/LAUNCHXL/release/mcuimg.bin -> /sys/factimg.bin [186112, disk=186112]...
..............................................
2020-02-13 21:15:34,499 -- Getting storage info...
2020-02-13 21:15:34,515 -- storage #2 info bytes: 0x10, 0x0, 0x1, 0x0, 0x0, 0x0, 0x0, 0x0
2020-02-13 21:15:34,515 -- Getting storage list...
2020-02-13 21:15:34,531 -- Reading raw storage #2 start 0x0, size 0x2000...
..
2020-02-13 21:15:34,659 -- [0] detected a valid FAT revision: 4
2020-02-13 21:15:34,659 -- selected FAT revision: 4 (active)
2020-02-13 21:15:34,659 -- FAT r4, num files: 2, used/free blocks: 54/202
2020-02-13 21:15:34,659 -- All commands done, bye.

```
書き込みが完了したらSO2はジャンパーを外し[000]にする。   
なお、cc3200toolを使用する際は、書き込み以外でもSO2は[100]にしないと動作しないようなので注意すること。


## 動作確認
(1)簡単な確認
```bash

picocom /dev/ttyUSB0 -b115200

MPY: soft reboot
MicroPython v1.12-167-gf020eac on 2020-02-13; LaunchPad with CC3200
Type "help()" for more information.
>>> 

>>> import os
>>> os.uname()
(sysname='WiPy', nodename='WiPy', release='1.2.0', version='v1.12-167-gf020eac on 2020-02-13', machine='LaunchPad with CC3200')
# sysname='WiPy'とあるので、WiPyボードと同等とみなされている
>>> 

>>> import gc
>>> gc.collect()
>>> gc.mem_free()
55376
>>> 

>>> 
>>> import sys
>>> sys.path
['', '/flash', '/flash/lib']
>>> 


>>> list(5 * x + y for x in range(10) for y in [4, 2, 1])
[4, 2, 1, 9, 7, 6, 14, 12, 11, 19, 17, 16, 24, 22, 21, 29, 27, 26, 34, 32, 31, 39, 37, 36, 44, 42, 41, 49, 47, 46]
>>> 

# 以下のようにfloatはサポートしていないので注意のこと
>>> 12+34
46
>>> 0.12+0.34
Traceback (most recent call last):
  File "<stdin>", line 1
SyntaxError: decimal numbers not supported
>>> 


```
MicroPythonから抜けるには、強引なやり方になるが、USBケーブルをいったん抜いて電源を切る。

(2)プログラムの実行

```bash

ampy put CC3200_starwars.py

picocom /dev/ttyUSB0 -b115200

import CC3200_starwars

```

実際に実行するプログラムは以下になる：  

以下の部分は自分のWiFi環境に合わせて変更すること：   
MYSSID = "your_ssid"   
MYKEY = "your_passwd"  

ＩＰを静的に与えるサンプルにもなっているので
IPがかち合っていれば、IPも変更すること。

CC3200_starwars.py
```python

MYSSID = "your_ssid"   
MYKEY = "your_passwd

import machine
from network import WLAN

# configure the WLAN subsystem in station mode (the default is AP)
wlan = WLAN(mode=WLAN.STA)
# go for fixed IP settings
wlan.ifconfig(config=('192.168.0.107', '255.255.255.0', '192.168.0.1', '8.8.8.8'))
#wlan.scan()     # scan for available networks
wlan.connect(ssid=MYSSID, auth=(WLAN.WPA2, MYKEY))
while not wlan.isconnected():
    pass
print(wlan.ifconfig())

#==========================================

# Star Wars ASCII Art (python3)

import socket
addr_info = socket.getaddrinfo("towel.blinkenlights.nl",23)
addr = addr_info[0][-1]

s = socket.socket()
s.connect(addr)

while True:
   data = s.recv(500)
   print(str(data, 'utf8'), end='')

#=============================================
```
starwarsのASCII_Artのアニメが表示されればＯＫとなる。

## ボードにプログラムを書き込む(自動起動になる)
書き込みたいプログラムをxxxx.pyとすると
以下のようにする：
```bash

ampy put xxxx.py /flash/main.py 
# 実行したいプログラムをmain.pyとして書き込む
# ボードの[reset]を押して起動する

#以下で書き込んであるプログラムが確認できる
ampy ls /flash
/flash/boot.py
/flash/cert
/flash/lib
/flash/main.py
/flash/sys
```

## コンソール(REPL)で実行したい場合
実行したいプログラムをCC3200_starwars.pyとすると
以下のようにする：

```bash

ampy put CC3200_starwars.py
picocom /dev/ttyUSB0 -b115200

import CC3200_starwars

```

## LEDs on board
MicroPythonから制御できるボード上のLEDとして以下の３つがある：  
・Red LED - D7(GP09)  
・Orange LED - D6(GP10)  
・Green LED  - D5(GP11)  

以下のように制御できる(REPL)：

```bash

picocom /dev/ttyUSB0 -b115200

# Red LED (hearbeat) on board D7(GP09)
import wipy
wipy.heartbeat(True)
wipy.heartbeat(False)
# Trueにすると4秒間隔で点滅する

from machine import Pin
# Orange LED - D6(GP10) on board
p_out = Pin('GP10', mode=Pin.OUT)
p_out.value(1)
p_out.value(0)
# Green LED  - D5(GP11) on board
p_out = Pin('GP11', mode=Pin.OUT)
p_out.value(1)
p_out.value(0)


```


## 参考情報 

[ＳｉｍｐｌｅＬｉｎｋ　Ｗｉ－Ｆｉ　ＣＣ３２００　Ｍｏｄｕｌｅ　ＬａｕｎｃｈＰａｄ](http://akizukidenshi.com/catalog/g/gM-09659/)  

[micropython/Quick reference for the WiP](https://docs.micropython.org/en/latest/wipy/quickref.html)  
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)　　　

[User:Dbartlett/MicroPython CC3200](https://processors.wiki.ti.com/index.php/User:Dbartlett/MicroPython_CC3200#Compiling_MicroPython)  
・役に立つ情報が多かったが、ここでのMicroPythonのバージョンが古いのでMicroPythonの部分は現状のものと異なる。  
[CC3200 Tool](https://github.com/ALLTERCO/cc3200tool)  


## 補足(書き込み直後のファイル状況)

/flashにあるファイル内容：
```bash

$ ampy ls /flash
/flash/boot.py
/flash/cert
/flash/lib
/flash/main.py
/flash/sys

$ ampy get /flash/boot.py
# boot.py -- run on boot-up
# can run arbitrary Python, but best to keep it minimal
import os, machine
os.dupterm(machine.UART(0, 115200))

$ ampy get /flash/main.py
# main.py -- put your code here!

# 以下のものは空だった
$ ampy ls /flash/cert
$ ampy ls /flash/lib
$ ampy ls /flash/sys

```

CC3200のシステム内部のファイル状況(/flashのものではない)
```bash

# ボードのジャンパーSOP2を[100]にすること  
$ cc3200tool -p /dev/ttyUSB0 list_filesystem
2020-02-14 09:48:23,790 -- Connecting to target...
2020-02-14 09:48:26,049 -- timed out while waiting for ack
2020-02-14 09:48:26,303 -- Connected, reading version...
2020-02-14 09:48:26,315 -- connected to target
2020-02-14 09:48:26,315 -- Version: CC3x00VersionInfo((0, 4, 1, 2), (0, 0, 0, 0), (0, 0, 0, 0), (0, 0, 0, 0), (16, 0, 0, 0))
2020-02-14 09:48:26,316 -- This is a CC3200 device
2020-02-14 09:48:26,316 -- Switching to NWP bootloader...
2020-02-14 09:48:26,330 -- Switching UART to APPS...
2020-02-14 09:48:26,346 -- Resetting communications ...
2020-02-14 09:48:27,609 -- Uploading rbtl3100s.dll...
2020-02-14 09:48:27,611 -- Getting storage list...
2020-02-14 09:48:27,625 -- Getting storage info...
2020-02-14 09:48:27,640 -- storage #0 info bytes: 0x10, 0x0, 0x0, 0x10, 0x0, 0x0, 0x0, 0x0
2020-02-14 09:48:28,360 -- APPS version: CC3x00VersionInfo((0, 4, 0, 2), (0, 0, 0, 0), (0, 0, 0, 0), (0, 0, 0, 0), (16, 0, 0, 0))
2020-02-14 09:48:28,360 -- Getting storage info...
2020-02-14 09:48:28,376 -- storage #2 info bytes: 0x10, 0x0, 0x1, 0x0, 0x0, 0x0, 0x0, 0x0
2020-02-14 09:48:28,376 -- Getting storage list...
2020-02-14 09:48:28,392 -- Reading raw storage #2 start 0x0, size 0x2000...
..
2020-02-14 09:48:28,520 -- [1] detected a valid FAT revision: 101
2020-02-14 09:48:28,520 -- selected FAT revision: 101 (active)
2020-02-14 09:48:28,521 -- Serial Flash block size:	4096 bytes
2020-02-14 09:48:28,521 -- Serial Flash capacity:	256 blocks
2020-02-14 09:48:28,522 -- 
2020-02-14 09:48:28,522 -- 	file	start	size	fail	flags	total size	filename
2020-02-14 09:48:28,522 -- 	index	block	[BLKs]	safe		[BLKs]
2020-02-14 09:48:28,522 -- ----------------------------------------------------------------------------
2020-02-14 09:48:28,522 -- 	N/A	0	5	N/A	N/A	5		FATFS
2020-02-14 09:48:28,522 -- 	0	5	3	no	0x4	3		/sys/mcuimg.bin
2020-02-14 09:48:28,522 -- 	4	8	46	no	0xc	46		/sys/factimg.bin
2020-02-14 09:48:28,523 -- 	6	54	5	yes	0x8	10		/tmp/phy.cal
2020-02-14 09:48:28,523 -- 	7	64	1	yes	0x8	2		/sys/bootinfo.bin
2020-02-14 09:48:28,523 -- 	8	66	2	yes	0x8	4		/sys/pref.net
2020-02-14 09:48:28,523 -- 	9	70	1	yes	0x8	2		/sys/ipcfg.ini
2020-02-14 09:48:28,523 -- 	10	72	1	yes	0x8	2		/sys/pmcfg.ini
2020-02-14 09:48:28,523 -- 	11	74	1	yes	0x8	2		/sys/mdns.cfg
2020-02-14 09:48:28,524 -- 	12	76	1	yes	0x8	2		/sys/mode.cfg
2020-02-14 09:48:28,524 -- 	13	78	1	yes	0x8	2		/sys/ap.cfg
2020-02-14 09:48:28,524 -- 	14	80	1	yes	0x8	2		/sys/dhcpsrv.cfg
2020-02-14 09:48:28,524 -- 	15	82	1	yes	0x8	2		/sys/date_time.cfg
2020-02-14 09:48:28,524 -- 	16	84	1	no	0xc	1		__000__.fsb
2020-02-14 09:48:28,524 -- 	17	85	1	no	0xc	1		__001__.fsb
...
省略
...
2020-02-14 09:48:28,528 -- 	47	114	1	no	0xc	1		__030__.fsb
2020-02-14 09:48:28,528 -- 	48	115	1	no	0xc	1		__031__.fsb
2020-02-14 09:48:28,529 -- 	49	116	1	yes	0x8	2		/sys/stacfg.ini
2020-02-14 09:48:28,529 -- 	50	118	1	yes	0x8	2		/tmp/fcon.frm
2020-02-14 09:48:28,529 -- 	51	120	1	yes	0x8	2		/tmp/fcon.ssid
2020-02-14 09:48:28,529 -- 
2020-02-14 09:48:28,529 --    Flash usage
2020-02-14 09:48:28,529 -- -------------------------
2020-02-14 09:48:28,529 -- used space:	122 blocks
2020-02-14 09:48:28,530 -- free space:	134 blocks
2020-02-14 09:48:28,530 -- memory hole:	[122-255]
2020-02-14 09:48:28,530 -- All commands done, bye.
# ボードのジャンパーSOP2を元に戻すこと([000]にすること)

```

以上
