
2020/2/2

MaixPy(MicroPython) WiFi test
# MaixPy(MicroPython) WiFi test

## 概要
Maixduinoに内蔵されているMicroPython(MaixPy)のWiFi動作確認について記する。WiFi機能についてはESP32が使われているので、MaixPyでESP32でアクセスすることになる。
(ホストPCとしてはubuntuを想定している)

## 事前準備
(1)ampyのインストール   

```bash

pip install adafruit-ampy
export AMPY_PORT=/dev/ttyUSB0
export AMPY_BAUD=115200
export AMPY_DELAY=0.5
```
重要：   
・他のMicroPythonボードと異なり「export AMPY_DELAY=0.5」が必要である。   
・シリアルポートが他のボートと異なり/dev/ttyACM0でなく/dev/ttyUSB0であることに注意。  

(2)picocomのインストール
```bash

sudo apt-get install picocom
```
以上のうち、exportしているものは、.bashrcに登録することを勧める。


## xxxx.pyの書き込みと実行方法

書き込み:
```bash

# sdに書き込む場合
ampy put xxxx.py /flash/xxxx.py

# flashに書き込む場合
ampy put xxxx.py /sd/xxxx.py

# デフォルトに書き込む場合
ampy put xxxx.py

```

実行:
```bash

picocom /dev/ttyUSB0 -b115200

# MaixPyのコンソールに入る
 __  __              _____  __   __  _____   __     __
|  \/  |     /\     |_   _| \ \ / / |  __ \  \ \   / /
| \  / |    /  \      | |    \ V /  | |__) |  \ \_/ /
| |\/| |   / /\ \     | |     > <   |  ___/    \   /
| |  | |  / ____ \   _| |_   / . \  | |         | |
|_|  |_| /_/    \_\ |_____| /_/ \_\ |_|         |_|

Official Site : https://www.sipeed.com
Wiki          : https://maixpy.sipeed.com

MicroPython v0.4.0-87-g42e92b235-dirty on 2019-11-07; Sipeed_M1 with kendryte-k210
Type "help()" for more information.
>>> 
# 実行プログラムの置き場所を選択する
import os
os.chdir('/flash')
#　または
os.chrdir('/sd')
# 内容の確認
os.listdir()
# xxxx.pyの実行
import xxxx
```
## WiFi確認用プログラム

MXD_ap_scan.py   
アクセスポイントを走査するプログラム
```python

import network
from Maix import GPIO
from fpioa_manager import fm, board_info

# IO map for ESP32 on Maixduino
fm.register(25,fm.fpioa.GPIOHS10)#cs
fm.register(8,fm.fpioa.GPIOHS11)#rst
fm.register(9,fm.fpioa.GPIOHS12)#rdy
fm.register(28,fm.fpioa.GPIOHS13)#mosi
fm.register(26,fm.fpioa.GPIOHS14)#miso
fm.register(27,fm.fpioa.GPIOHS15)#sclk

nic = network.ESP32_SPI(cs=fm.fpioa.GPIOHS10,rst=fm.fpioa.GPIOHS11,rdy=fm.fpioa.GPIOHS12, mosi=fm.fpioa.GPIOHS13,miso=fm.fpioa.GPIOHS14,sclk=fm.fpioa.GPIOHS15)

# bug fix
#enc_str = ["OPEN", "", "WPA PSK", "WPA2 PSK", "WPA/WPA2 PSK"]
enc_str = ["OPEN", "", "WPA PSK", "WPA2 PSK", "WPA/WPA2 PSK","WEP"]
aps = nic.scan()
for ap in aps:
    print("SSID:{:^20}, ENC:{:>5} , RSSI:{:^20}".format(ap[0], enc_str[ap[1]], ap[2]) )

```
実行例：
```bash

>>> import MXD_ap_scan
SSID:2C1B48970BBA6A47473A8E215865F5F3, ENC:WPA PSK , RSSI:        -58         
SSID:5DD34D5E34AE91F443EDF9D27012935E, ENC:  WEP , RSSI:        -58         
SSID:    001D731D0A98    , ENC:WPA/WPA2 PSK , RSSI:        -59         
...
省略
...

>>> 

```


MXD_starwars.py   
StarWarsのASCII_Artを表示するプログラム   
以下は自分の環境に合わせて変更すること:   
WIFI_SSID = "your_ssid"  
WIFI_PASSWD = "your passwd"  
```python

# NIC setup
import network, socket
from Maix import GPIO
from fpioa_manager import fm, board_info

WIFI_SSID = "your_ssid"
WIFI_PASSWD = "your passwd"

# IO map for ESP32 on Maixduino
fm.register(25,fm.fpioa.GPIOHS10)#cs
fm.register(8,fm.fpioa.GPIOHS11)#rst
fm.register(9,fm.fpioa.GPIOHS12)#rdy
fm.register(28,fm.fpioa.GPIOHS13)#mosi
fm.register(26,fm.fpioa.GPIOHS14)#miso
fm.register(27,fm.fpioa.GPIOHS15)#sclk

nic = network.ESP32_SPI(cs=fm.fpioa.GPIOHS10,rst=fm.fpioa.GPIOHS11,rdy=fm.fpioa.GPIOHS12, mosi=fm.fpioa.GPIOHS13,miso=fm.fpioa.GPIOHS14,sclk=fm.fpioa.GPIOHS15)

print("ESP32_SPI firmware version:", nic.version())

err = 0
while 1:
    try:
        nic.connect(WIFI_SSID, WIFI_PASSWD)
    except Exception:
        err += 1
        print("Connect AP failed, now try again")
        if err > 3:
            raise Exception("Conenct AP fail")
        continue
    break
print(nic.ifconfig())
print(nic.isconnected())

err = 0

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
途中でエラーで止まるようだが、WiFiアクセスの確認には使用できる。　　　   
なお、「Star Wars ASCII Art (python3)」のコメント行以下は
ＰＣのpython3で実行可能なので、「setup NIC」以外の部分を
ＰＣのpython3で動作確認することはネットワーク関係については可能だと思われる。

実行例：
```python

>>>import MXD_starwars
                            
      Original Work   : Simon Jansen ( http://www.asciimation.co.nz/ )   
      Telnetification : Sten Spans ( http://blinkenlights.nl/ )          
      Terminal Tricks : Mike Edwards (pf-asciimation@mirkwood.net)       
                                                                         
      The hard work was done by Simon and Mike,                          
      I just placed it online




                                           /~\                           
                                          ( oo|    They've shut down     
                                          _\=/_    the main reactor.     
                          ___         #  /  _  \                         
                         / ()\        \\//|/.\|\\                        
                       _|_____|_       \/  \_/  ||                       
                      | | === | |         |\ /| ||                       
                      |_|  O  |_|         \_ _/   #                      
                       ||  O  ||          | | |                          
                       ||__*__||          | | |                          
                      |~ \___/ ~|         []|[]                          
                      /=\ /=\ /=\         | | |                          
      ________________[_]_[_]_[_]________/_]_[_\___________________



```



## 参照URL

[Maixduino(schematic)](http://dl.sipeed.com/MAIX/HDK/Maixduino/Maixduino-4.30/Maixduino-4.30%28schematic%29.pdf)   
[K210 Deep Learning MCU #51](https://github.com/NirViaje/nirviaje.github.io/issues/51)   
[MaixPy Documentation](https://maixpy.sipeed.com/en/)   
[The MicroPython project](https://github.com/micropython/micropython)     
[Quick reference for the pyboard](http://docs.micropython.org/en/latest/pyboard/quickref.html)   
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)    
[MicroPython pyboard v1.1](https://www.switch-science.com/catalog/3488/)   
[Sipeed Maixduino](https://www.switch-science.com/catalog/5707/)   

以上
