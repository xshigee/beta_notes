
2020/2/9

ESP32 MicroPython OSC send/recv
# ESP32 MicroPython OSC send/recv

## 概要
ESP32のMicroPythonでOSC(Open Sound Control)で通信する方法について述べる。受信のほうは、足りない内蔵モジュールがあるせいでエラーして動かなかった。エラー回避のためにOSC_messageのデコーダ専用のOSCmsg.pyを作り、それをインストールした。
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

## エラー回避した追加モジュールのインストール

```bash
# 追加モジュールのインストール
ampy put socketutil.py /lib/uosc/OSCmsg.py  
# インストールしたモジュールの確認
ampy ls /lib/uosc
/lib/uosc/OSCmsg.py
...
```

実際のソース：  
OSCmsg.py   
(server.pyからエラー回避のためにlogging関連のコードなどを削除したもの)
```python

# -*- coding: utf-8 -*-
#
#  uosc/OSCmsg.py
#  2020/2/9: forked from server.py
#
""" OSCmsg """

from ustruct import unpack
from uosc.common import Impulse, to_time

MAX_DGRAM_SIZE = 1472


def split_oscstr(msg, offset):
    end = msg.find(b'\0', offset)
    return msg[offset:end].decode('utf-8'), (end + 4) & ~0x03


def split_oscblob(msg, offset):
    start = offset + 4
    size = unpack('>I', msg[offset:start])[0]
    return msg[start:start + size], (start + size + 4) & ~0x03


def parse_timetag(msg, offset):
    """Parse an OSC timetag from msg at offset."""
    return to_time(unpack('>II', msg[offset:offset + 4]))


def parse_message(msg, strict=False):
    args = []
    addr, ofs = split_oscstr(msg, 0)

    if not addr.startswith('/'):
        raise ValueError("OSC address pattern must start with a slash.")

    # type tag string must start with comma (ASCII 44)
    if ofs < len(msg) and msg[ofs:ofs + 1] == b',':
        tags, ofs = split_oscstr(msg, ofs)
        tags = tags[1:]
    else:
        errmsg = "Missing/invalid OSC type tag string."
        if strict:
            raise ValueError(errmsg)
        else:
            tags = ''

    for typetag in tags:
        size = 0

        if typetag in 'ifd':
            size = 8 if typetag == 'd' else 4
            args.append(unpack('>' + typetag, msg[ofs:ofs + size])[0])
        elif typetag in 'sS':
            s, ofs = split_oscstr(msg, ofs)
            args.append(s)
        elif typetag == 'b':
            s, ofs = split_oscblob(msg, ofs)
            args.append(s)
        elif typetag in 'rm':
            size = 4
            args.append(unpack('BBBB', msg[ofs:ofs + size]))
        elif typetag == 'c':
            size = 4
            args.append(chr(unpack('>I', msg[ofs:ofs + size])[0]))
        elif typetag == 'h':
            size = 8
            args.append(unpack('>q', msg[ofs:ofs + size])[0])
        elif typetag == 't':
            size = 8
            args.append(parse_timetag(msg, ofs))
        elif typetag in 'TFNI':
            args.append({'T': True, 'F': False, 'I': Impulse}.get(typetag))
        else:
            raise ValueError("Type tag '%s' not supported." % typetag)

        ofs += size

    return (addr, tags, tuple(args))


def parse_bundle(bundle, strict=False):
    """Parse a binary OSC bundle.

    Returns a generator which walks over all contained messages and bundles
    recursively, depth-first. Each item yielded is a (timetag, message) tuple.

    """
    if not bundle.startswith(b'#bundle\0'):
        raise TypeError("Bundle must start with b'#bundle\\0'.")

    ofs = 16
    timetag = to_time(*unpack('>II', bundle[8:ofs]))

    while True:
        if ofs >= len(bundle):
            break

        size = unpack('>I', bundle[ofs:ofs + 4])[0]
        element = bundle[ofs + 4:ofs + 4 + size]
        ofs += size + 4

        if element.startswith(b'#bundle'):
            for el in parse_bundle(element):
                yield el
        else:
            yield timetag, parse_message(element, strict)


def handle_osc(data, src, dispatch=None, strict=False):
    try:
        head, _ = split_oscstr(data, 0)

        if head.startswith('/'):
            messages = [(-1, parse_message(data, strict))]
        elif head == '#bundle':
            messages = parse_bundle(data, strict)
    except Exception as exc:
        return

    try:
        for timetag, (oscaddr, tags, args) in messages:
            if dispatch:
                dispatch(timetag, (oscaddr, tags, args, src))
    except Exception as exc:
        print("Exception in OSC handler: %s", exc)



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

## 受信テスト
送信側になるTouchOSCを起動してレイアウトSimpleのＰａｎｅｌ#1を表示して
待ち受けにして以下のコマンドを実行する：
```bash

# プログラムを書き込む
ampy put ESP32_OSCrecvBundled_test.py

picocom /dev/ttyUSB0 -b115200
# REPLに入る

import ESP32_OSCrecvBundled_test.py
# WiFiが接続されIPアドレスが表示される
# そのIPアドレスをTouchOSCのHostのIPアドレスとして設定する
# TouchOSCの画面のfaderやtoggleを動かすと
# そのOSCパケットを受信してREPL画面に表示される

```

実際のソース：  
ESP32_OSCrecvBundled_test.py
```python
# handling bundled message
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
s.setblocking(True)

while True:   
   data,addr=s.recvfrom(1024)
   print('received:',data,'from',addr)
   head, _ = split_oscstr(data, 0)
   if head.startswith('/'):
      msg = parse_message(data)
      print(msg)
   elif head == '#bundle':
      for x in parse_bundle(data):
         timetag, msg = x
         print(msg)
   print('=======================') 


```

REPL出力例：
```
myIP:192.168.0.14
listening...
received: b'/1/fader2\x00\x00\x00,f\x00\x00?\x05\xf0\xe0' from ('192.168.0.2', 9000)
('/1/fader2', 'f', (0.5232067,))
=======================
received: b'/1/fader4\x00\x00\x00,f\x00\x00?E!|' from ('192.168.0.2', 9000)
('/1/fader4', 'f', (0.7700422,))
=======================
received: b'/1/fader5\x00\x00\x00,f\x00\x00>\x84HT' from ('192.168.0.2', 9000)
('/1/fader5', 'f', (0.2583643,))
=======================
received: b'/1/fader5\x00\x00\x00,f\x00\x00>\x98D\x85' from ('192.168.0.2', 9000)
('/1/fader5', 'f', (0.2973978,))
=======================

# 以降、TouchOSCの「Option/Bundle Messages」をオンにする

received: b'#bundle\x00\xe1\xeb1\x93=[W>\x00\x00\x00\x14/1/fader2\x00\x00\x00,f\x00\x00?9\xc9\xfe' from ('192.168.0.2', 9000)
('/1/fader2', 'f', (0.7257384,))
=======================
received: b'#bundle\x00\xe1\xeb1\x94\x9b\xa7\xda\xa4\x00\x00\x00\x14/1/fader4\x00\x00\x00,f\x00\x00>\x98M\xc6' from ('192.168.0.2', 9000)
('/1/fader4', 'f', (0.2974684,))
=======================
received: b'#bundle\x00\xe1\xeb1\x94\xb8P\x04\xfa\x00\x00\x00\x14/1/fader4\x00\x00\x00,f\x00\x00>\xab\xbf0' from ('192.168.0.2', 9000)
('/1/fader4', 'f', (0.335443,))
=======================

```

デコーダ関数parse_messageの値の返し方
```python

oscaddr, tag, values = parse_message(data)
>>> oscaddr
'/1/fader1'
>>> tag
'f'
>>> values
(0.7362869,)
>>> values[0]
0.7362869
>>> 

>>> oscmsg = parse_message(data)
>>> oscmsg
('/1/fader1', 'f', (0.7362869,))
>>> oscmsg[0]
'/1/fader1'
>>> oscmsg[1]
'f'
>>> oscmsg[2]
(0.7362869,)
>>> oscmsg[2][0]
0.7362869
>>> 

```

## 参考情報  

[TouchOSC | Control reference](https://hexler.net/docs/touchosc-controls-reference)  
[OpenSound Control](https://ja.wikipedia.org/wiki/OpenSound_Control)  

[ＥＳＰ３２－ＤｅｖＫｉｔＣ　ＥＳＰ－ＷＲＯＯＭ－３２開発ボード](http://akizukidenshi.com/catalog/g/gM-11819/)    
[MicroPython - Quick reference for the ESP32](https://docs.micropython.org/en/latest/esp32/quickref.html#)    
[Streaming Data from ESP32 using MicroPython and MQTT](https://github.com/gloveboxes/ESP32-MicroPython-BME280-MQTT-Sample/blob/master/Micropython.md)   
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)　　　

以上
