
2020/2/9

ESP32 MicroPython OSCsend
# ESP32 MicroPython OSCsend

## 概要
ESP32のMicroPythonでOSC(Open Sound Control)で送信する方法について述べる。(受信のほうは、足りないモジュールがあるせいかエラーして動かなかった。)
ここでは、linux環境でのインストール方法について説明する。   

## 準備
ツールをインストール前に環境整備として以下を設定する：  
(1)書き込みツール(esptool)のインストール   
以下のURLの参照のこと；  
[esptool.py](https://github.com/espressif/esptool)  

(この記事のなかでは使用しない)   

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

## 受信側アプリの準備
ここでは、iPhoneのTouchOSCを使用する。   
以下、インストール用URL：  
[iPhone/iPad TouchOSC](https://apps.apple.com/app/touchosc/id288120394)   
[Android TouchOSC](https://play.google.com/store/apps/details?id=net.hexler.touchosc_a)   


## WiFi接続モジュールのインストール
```bash

ampy put wlan.py
```

wlan.py
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

## OSCモジュールのインストール
本来ならば以下をREPLで実行することで該当モジュールのインストールができるはずだが
sslでエラーが出るようでインストールできなかった。

```python

# connect wlan
SSID="your_ssid"
PASSWORD="your_password"

from wlan import wlan_connect
wlan_connect(SSID, PASSWORD)

import uos
import uos.mkdir("/lib")

#--------------------------
import upip
upip.install("micropython-osc")

```

そこで以下の方法でインストールした：   
(ホストPC上で以下を行なう)
```bash
#　OSCモジュールのインストール
git clone https://github.com/SpotlightKid/micropython-osc.git
cd micropython-osc/uosc
ampy mkdir /lib
ampy mkdir /lib/uosc
ampy put client.py /lib/uosc/client.py  
ampy put server.py /lib/uosc/server.py    
ampy put threadedclient.py /lib/uosc/threadedclient.py
ampy put common.py /lib/uosc/common.py  
ampy put socketutil.py /lib/uosc/socketutil.py  
# インストールしたモジュールの確認
ampy ls /lib/uosc
/lib/uosc/client.py
/lib/uosc/common.py
/lib/uosc/server.py
/lib/uosc/socketutil.py
/lib/uosc/threadedclient.py
```

## 送信テスト
受信側になるTouchOSCを起動してレイアウトSimpleのＰａｎｅｌ#1を表示して
待ち受けにして以下のコマンドを実行する：
```bash

ampy run ESP32_OSCsend_test.py


```
実行するとOSCのパケットがTouchOSCに届き、TouchOSC画面のfaderやtoggleが動けば
テストＯＫになる。  


使用したプログラムは以下になる：  
ESP32_OSCsend_test.py  

以下の部分は自分のWiFi環境に合わせて変更すること：   
SSID = "your_ssid"   
PASSWORD = "your_passwd"    

以下は、TouchOSCの設定に合わせて修正する：  
IP='192.168.0.20' # TouchOSCのLocal IP address  
OUT_PORT=9000　# TouchOSCのPort(incomming)
```python

# connect wlan
SSID = "your_ssid"   
PASSWORD = "your_passwd"  [

from wlan import wlan_connect
wlan_connect(SSID, PASSWORD)

#-------------------------------------
# test for TouchOSC(Layout:Simple)
from uosc.client import Bundle, Client, create_message

IP='192.168.0.20'
OUT_PORT=9000

osc = Client(IP, OUT_PORT)

# panel#1
# fader#1
for n in range(1,100):
   osc.send('/1/fader1', n/100)
for n in range(1,100):
   osc.send('/1/fader1', 1-n/100)
# fader#2
for n in range(1,100):
   osc.send('/1/fader2', n/100)
for n in range(1,100):
   osc.send('/1/fader2', 1-n/100)
# fader#3
for n in range(1,100):
   osc.send('/1/fader3', n/100)
for n in range(1,100):
   osc.send('/1/fader3', 1-n/100)
# fader#4
for n in range(1,100):
   osc.send('/1/fader4', n/100)
for n in range(1,100):
   osc.send('/1/fader4', 1-n/100)
# fader#5
for n in range(1,100):
   osc.send('/1/fader5', n/100)
for n in range(1,100):
   osc.send('/1/fader5', 1-n/100)
# toggles
osc.send('/1/toggle1', 1)
osc.send('/1/toggle2', 1)
osc.send('/1/toggle3', 1)
osc.send('/1/toggle4', 1)
osc.send('/1/toggle1', 0)
osc.send('/1/toggle2', 1)
osc.send('/1/toggle3', 0)
osc.send('/1/toggle4', 1)
#----------------------------------
# end of panel#1

```
「ampy run」でエラーになって動作しない場合、同じプログラムをREPLでコピー＆ペーストして実行すると動作することがある。


## 参考情報  

[TouchOSC | Control reference](https://hexler.net/docs/touchosc-controls-reference)  
[OpenSound Control](https://ja.wikipedia.org/wiki/OpenSound_Control)  

[ＥＳＰ３２－ＤｅｖＫｉｔＣ　ＥＳＰ－ＷＲＯＯＭ－３２開発ボード](http://akizukidenshi.com/catalog/g/gM-11819/)    
[MicroPython - Quick reference for the ESP32](https://docs.micropython.org/en/latest/esp32/quickref.html#)    
[Streaming Data from ESP32 using MicroPython and MQTT](https://github.com/gloveboxes/ESP32-MicroPython-BME280-MQTT-Sample/blob/master/Micropython.md)   
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)　　　

以上
