
2020/2/17

CC3200 MicroPython OSC recv
# CC3200 MicroPython OSC recv

## 概要
CC3200のMicroPythonでOSC(Open Sound Control)で受信する方法について述べる。これはESP32のもの同様のものであるが、CC3200の場合、floatが実装されていないので、そこが差分になる。float32は、符号1ビット、指数部8ビット、仮数部23ビットで構成されるが、その32ビットが、そのまま整数32ビットに読み替えられる。

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

## 送信側アプリの準備
ここでは、iPhoneのTouchOSCを使用する。   
以下、インストール用URL：  
[iPhone/iPad TouchOSC](https://apps.apple.com/app/touchosc/id288120394)   
[Android TouchOSC](https://play.google.com/store/apps/details?id=net.hexler.touchosc_a)   


## OSCモジュールのインストール

以下の方法でインストールした：   
(ホストPC上で以下を行なう)
```bash
#　OSCモジュールのインストール
git clone https://github.com/SpotlightKid/micropython-osc.git
cd micropython-osc/uosc
ampy mkdir /flash/lib
ampy mkdir /flash/lib/uosc
ampy put client.py /flash/lib/uosc/client.py  
ampy put server.py /flash/lib/uosc/server.py    
ampy put threadedclient.py /flash/lib/uosc/threadedclient.py
ampy put common.py /flash/lib/uosc/common.py  
ampy put socketutil.py /flash/lib/uosc/socketutil.py  
# インストールしたモジュールの確認
ampy ls /flash/lib/uosc
/flash/lib/uosc/client.py
/flash/lib/uosc/common.py
/flash/lib/uosc/server.py
/flash/lib/uosc/socketutil.py
/flash/lib/uosc/threadedclient.py
```

## エラー回避した追加モジュールのインストール

```bash
# 追加モジュールのインストール
ampy put socketutil.py /flash/lib/uosc/OSCmsg.py  
# インストールしたモジュールの確認
ampy ls /flash/lib/uosc
/flash/lib/uosc/OSCmsg.py
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



## 受信テスト
送信側になるTouchOSCを起動してレイアウトSimpleのPanel#1を表示して
待ち受けにして以下のコマンドを実行する：
```bash

# プログラムを書き込む
ampy put CC3200_OSCrecvBundled_test.py

picocom /dev/ttyUSB0 -b115200
# REPLに入る

import CC3200_OSCrecvBundled_test.py
# WiFiが接続されIPアドレスが表示される
# そのIPアドレスをTouchOSCのHostのIPアドレスとして設定する
# TouchOSCの画面のfaderやtoggleを動かすと
# そのOSCパケットを受信してREPL画面に表示される

```

実際のソース：  
CC3200_OSCrecvBundled_test.py  
以下の部分を自分の環境に合わせる；  
MYSSID="your_ssid"  
MYKEY="your_passwd"  
   
```python


# CC3200 WiFi setup
MYSSID="your_ssid"
MYKEY="your_passwd"

import machine
from network import WLAN

# configure the WLAN subsystem in station mode (the default is AP)
wlan = WLAN(mode=WLAN.STA)
# go for fixed IP settings
#wlan.ifconfig(config=('192.168.0.107', '255.255.255.0', '192.168.0.1', '8.8.8.8'))
#wlan.scan()     # scan for available networks
wlan.connect(ssid=MYSSID, auth=(WLAN.WPA2, MYKEY))
while not wlan.isconnected():
    pass
print(wlan.ifconfig())

#-----------------------------
from uosc.OSCmsg import parse_message, parse_bundle, split_oscstr
import usocket as socket
import network

incoming_port = 8000

# get my IP
myIP, netmask, gateway, dns = wlan.ifconfig()

s=socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
#s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1) 
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
# '#bundle' does not work 
#   elif head == '#bundle':
#      for x in parse_bundle(data):
#         timetag, msg = x
#         print(msg)
   print('=======================') 


```

REPL出力例：
```
>>> import CC3200_OSCrecvBundled_test
('192.168.0.23', '255.255.255.0', '192.168.0.1', '192.168.0.1')
myIP:192.168.0.23
listening...
received: b'/1/fader1\x00\x00\x00,f\x00\x00\x00\x00\x00\x00' from ('192.168.0.16', 9000)
('/1/fader1', 'f', (0,))
#                 0 は、 0.0 に対応している
=======================
received: b'/1/fader5\x00\x00\x00,f\x00\x00?\x80\x00\x00' from ('192.168.0.16', 9000)
('/1/fader5', 'f', (1065353216,))
#                 1065353216 は、 1.0 に対応している
=======================

```


## 参考情報  

[TouchOSC | Control reference](https://hexler.net/docs/touchosc-controls-reference)  
[OpenSound Control](https://ja.wikipedia.org/wiki/OpenSound_Control)  
[ESP32のMicroPythonでOSC(Open Sound Control)で通信する](https://beta-notes.way-nifty.com/blog/2020/02/post-15dff0.html)  

[CC3200 MicroPython examples](https://beta-notes.way-nifty.com/blog/2020/02/post-dac03e.html)  
[CC3200 MicroPython インストール](https://beta-notes.way-nifty.com/blog/2020/02/post-33f0ab.html)  
[micropython - Quick reference for the WiPy](https://docs.micropython.org/en/latest/wipy/quickref.html)  
  
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)　　　

以上
