
2020/2/11+

M5Stack Customized MicroPython Install
# M5Stack Customized MicroPython Install

## 概要
ESP32のMicroPythonのfirmwareは色々なものがあり、どれが良いか悩むことになるが。ここでは、以下のURLにあったカスタマイズしたMicroPythonをM5Stackにインストールしてみる。内容はオリジナルの記事とほとんど同じだが、windows環境ではなく、ubuntu環境でのやり方を説明する。   

[m5stackでmicropythonをつかう(SDカードブートも)](https://www.yukkuriikouze.com/2019/05/11/2857/#toc_id_2_3)   

## 準備
ツールをインストール前に環境整備として以下を設定する：  
(1)書き込みツール(esptool)のインストール   
以下のURLの参照のこと；  
[esptool.py](https://github.com/espressif/esptool)  

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

## firmwareのダウンロードと書き込み
以下の手順で実行する：
```bash

mkdir M5Stack
cd M5Stack
wget https://www.yukkuriikouze.com/wp-content/uploads/2019/05/m5stack_micropython.zip
unzip m5stack_micropython.zip

# unzipした内容の確認
ls
firmware.bin  log.txt  sd_boot  sd_boot_firmware.bin

# firmwareの書き込み
# (ここではSDブートするバージョンを採用した)
esptool.py --port /dev/ttyUSB0 --baud 460800 write_flash 0 sd_boot_firmware.bin
#ログ出力例：
esptool.py v2.8
Serial port /dev/ttyUSB0
Connecting....
Detecting chip type... ESP32
Chip is ESP32D0WDQ6 (revision 1)
Features: WiFi, BT, Dual Core, Coding Scheme None
Crystal is 40MHz
MAC: 30:ae:a4:58:7c:f0
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 460800
Changed.
Configuring flash size...
Auto-detected Flash size: 16MB
Compressed 16777216 bytes to 1325927...
Wrote 16777216 bytes (1325927 compressed) at 0x00000000 in 72.4 seconds (effective 1853.3 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...

```
以上でfirmwareの書き込みが終了する。

## sdmain.py
SDブートするプログラムは、sdmain.pyの名前でSDにファイルとして置く。(SDのトップに置く)   

内容は以下：  
sdmain.py
```python

import sys
# Set default path
# Needed for importing modules and upip
sys.path[1] = '/sd/lib'
#
from m5stack import lcd
lcd.print('Booting at ')
import os
lcd.print(os.getcwd())
lcd.print('\n')
#
#-----------------------
# go another module
import M5S_OSCrecvBundled_test # ここでSDに置いてある実行ファイルを指定する

```

## M5S_OSCrecvBundled_test.py
M5S_OSCrecvBundled_test.py (SDのトップに置く)    
これは以下のURLで紹介したもののM5Stack版となる。   

[ESP32のMicroPythonでOSC(Open Sound Control)で通信する](https://beta-notes.way-nifty.com/blog/2020/02/post-15dff0.html)  

以下は、自分のWiFi環境に合わせて変更する：  
SID="your_ssid"  
PASSWORD="your_password"  
  
```python

# handling bundled message
from m5stack import lcd
# connect wlan
SSID="your_ssid"
PASSWORD="your_password"

from wlan import wlan_connect
wlan_connect(SSID, PASSWORD)

#-----------------------------
from uosc.OSCmsg import parse_message, parse_bundle, split_oscstr
import usocket as socket
import network

incoming_port = 8000

sta_if = network.WLAN(network.STA_IF)
myIP, netmask, gateway, dns = sta_if.ifconfig()

s=socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1) 
s.bind(('0.0.0.0',incoming_port)) 

print('myIP:'+myIP)
print('listening...')
lcd.clear()
lcd.print('',0,0)
lcd.print('myIP:'+myIP+'\n')
lcd.print('listening...'+'\n')
s.setblocking(True)

maxlin = 14
linum = 1
while True:   
   data,addr=s.recvfrom(1024)
   print('received:',data,'from',addr)
   head, _ = split_oscstr(data, 0)
   if head.startswith('/'):
      msg = parse_message(data)
      print(msg)
      smsg = str(msg)
      if (linum % maxlin)==0:
         lcd.clear()
         lcd.print('',0,0)
      lcd.print(smsg+'\n')
      linum += 1
   elif head == '#bundle':
      for x in parse_bundle(data):
         timetag, msg = x
         print(msg)
         smsg = str(msg)
         if (linum % maxlin)==0:
            lcd.clear()
            lcd.print('',0,0)
         lcd.print('#bundle: '+smsg+'\n')
         linum += 1
   print('=======================')
 
#---------------------------------

```

## その他のSDに置く.pyファイル
wlan.py (SDのトップに置く) 
```python

# wlan.py
def wlan_connect(ssid='SSID', password='PASSWD'):
    import network
    wlan = network.WLAN(network.STA_IF)
    if not wlan.active() or not wlan.isconnected():
        wlan.active(True)
        print('connecting to:', ssid)
        wlan.connect(ssid, password)
        while not wlan.isconnected():
            pass

```

OSCモジュール:  

sdに/lib/uoscのデイレクトリを作り、以下の記事で説明しているpyファイルをそこに置く。  
[ESP32のMicroPythonでOSC(Open Sound Control)で通信する](https://beta-notes.way-nifty.com/blog/2020/02/post-15dff0.html)   
```bash
結果として以下のファイルを置く：  
/lib/uosc/client.py  
/lib/uosc/server.py    
/lib/uosc/threadedclient.py
/lib/uosc/common.py  
/lib/uosc/socketutil.py  
/lib/uosc/OSCmsg.py  
```

## OSC受信テスト
上で説明したSDができたら、そのSDをM5Stackに入れてブートすると
OSC受信テストプログラムが起動する。  
M5StackのIPアドレスがM5StackのLCDに表示されるので
それをiPhoneの送信側になるTouchOSCのHostのIPアドレスとして設定する。
TouchOSCのレイアウトSimpleのＰａｎｅｌ#1を表示してfaderやtoggleを動かすと
そのOSCパケットの内容がM5StackのLCDに表示される。

## 参考情報  

[m5stackでmicropythonをつかう(SDカードブートも)](https://www.yukkuriikouze.com/2019/05/11/2857/#toc_id_2_3)   
[M5Stack用MicroPythonのビルドとカスタマイズ](https://qiita.com/ciniml/items/1378d02bc14098b959ef)  
  
[TouchOSC | Control reference](https://hexler.net/docs/touchosc-controls-reference)  
[OpenSound Control](https://ja.wikipedia.org/wiki/OpenSound_Control)  
  
[ＥＳＰ３２－ＤｅｖＫｉｔＣ　ＥＳＰ－ＷＲＯＯＭ－３２開発ボード](http://akizukidenshi.com/catalog/g/gM-11819/)    
[MicroPython - Quick reference for the ESP32](https://docs.micropython.org/en/latest/esp32/quickref.html#)    
[Streaming Data from ESP32 using MicroPython and MQTT](https://github.com/gloveboxes/ESP32-MicroPython-BME280-MQTT-Sample/blob/master/Micropython.md)   
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)　　　


## /dev/ttyUSB0消失問題
ホストPCに接続しても/dev/ttyUSB0が出てこないで、手も足もでない状態になったことがあった。推測だがM5StackのUSBシリアル回路のリセットに問題があるらしい。復旧方法として、M5Stackの電源を入れっぱなしにして（一晩くらい）バッテリを空にしてから、USBを刺して電源を入れ直すと復旧する。（バッテリ空状態で電源を入れると正常にリセットがかかる）

## PSRAMサポート
firmwareでPSRAMを自動検出して、あればPSRAMをサポートするようになっている。ちなみに私が使っているM5StackにはPSRAMは付いていないようだ。(boot時のメッセージで分かる)  
「M5Stack FIRE」は、以下のリンクによるとPSRAM付きのようだ。  
[M5Stack FIRE](https://www.switch-science.com/catalog/3953/?gclid=EAIaIQobChMIiby3hPHk5gIVmHZgCh2XOQ-uEAAYASAAEgKzhfD_BwE)  


以上
