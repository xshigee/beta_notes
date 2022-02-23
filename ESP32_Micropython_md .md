
2020/2/6

ESP32 MicroPython Install
# ESP32 MicroPython Install

## 概要
「ＥＳＰ３２－ＤｅｖＫｉｔＣ　ＥＳＰ－ＷＲＯＯＭ－３２開発ボード(ESP32)」にMicroPythonのfirewareを書き込む方法について述べる。
ここでは、linux環境でのインストール方法について説明する。   

## 準備
ツールをインストール前に環境整備として以下を設定する：  
(1)esp-open-sdkのインストール   
firewareの書き込みだけでは、sdkは不要だが、今後のことを考えてsdkをインストールする。
```bash

sudo apt-get install make unrar-free autoconf automake libtool gcc g++ gperf flex bison texinfo gawk ncurses-dev libexpat-dev python-dev python python-serial sed git unzip bash help2man wget bzip2 libtool-bin

mkdir mp
cd mp
git clone --recursive https://github.com/pfalcon/esp-open-sdk.git

cd esp-open-sdk
make
# 30分以上、時間がかかるので、ひたすら、待つ。

# sdkのディレクトリにパスを通す
export PATH=~/mp/esp-open-sdk/xtensa-lx106-elf/bin:$PATH

```

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

## firewareのダウンロード＆書き込み
```bash

mkdir esp32
cd esp32

wget https://micropython.org/resources/firmware/esp32-idf3-20200117-v1.12-68-g3032ae115.bin

# ボードをホストＰＣに接続しておく
# flashを消去する
esptool.py --chip esp32 --port /dev/ttyUSB0 erase_flash
# 出力ログ例
esptool.py v2.8
Serial port /dev/ttyUSB0
Connecting....
Chip is ESP32D0WDQ6 (revision 0)
Features: WiFi, BT, Dual Core, Coding Scheme None
Crystal is 40MHz
MAC: 30:ae:a4:03:88:a8
Uploading stub...
Running stub...
Stub running...
Erasing flash (this may take a while)...
Chip erase completed successfully in 2.1s
Hard resetting via RTS pin...

# firmwareを書き込む

esptool.py --chip esp32 --port /dev/ttyUSB0 --baud 460800 write_flash -z 0x1000 esp32-idf3-20200117-v1.12-68-g3032ae115.bin 
# 出力ログ例
esptool.py v2.8
Serial port /dev/ttyUSB0
Connecting.....
Chip is ESP32D0WDQ6 (revision 0)
Features: WiFi, BT, Dual Core, Coding Scheme None
Crystal is 40MHz
MAC: 30:ae:a4:03:88:a8
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 460800
Changed.
Configuring flash size...
Auto-detected Flash size: 4MB
Compressed 1433568 bytes to 912834...
Wrote 1433568 bytes (912834 compressed) at 0x00001000 in 22.6 seconds (effective 507.4 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...
```

## 動作確認
(1)簡単な確認
```bash

picocom /dev/ttyUSB0 -b115200

ログ出力：
>>> list(5 * x + y for x in range(10) for y in [4, 2, 1])
[4, 2, 1, 9, 7, 6, 14, 12, 11, 19, 17, 16, 24, 22, 21, 29, 27, 26, 34, 32, 31, 39, 37, 36, 44, 42, 41, 49, 47, 46]
>>>　Ctrl-D 

>>> 
MPY: soft reboot
MicroPython v1.12-68-g3032ae115 on 2020-01-17; ESP32 module with ESP32
Type "help()" for more information.
>>> 

# 内蔵モジュールの表示
>>> help('modules')
__main__          gc                uctypes           urequests
_boot             inisetup          uerrno            uselect
_onewire          machine           uhashlib          usocket
_thread           math              uhashlib          ussl
_webrepl          micropython       uheapq            ustruct
apa106            neopixel          uio               utime
btree             network           ujson             utimeq
builtins          ntptime           umqtt/robust      uwebsocket
cmath             onewire           umqtt/simple      uzlib
dht               sys               uos               webrepl
ds18x20           uarray            upip              webrepl_setup
esp               ubinascii         upip_utarfile     websocket_helper
esp32             ubluetooth        upysh
flashbdev         ucollections      urandom
framebuf          ucryptolib        ure
Plus any modules on the filesystem
>>> 


```
MicroPythonから抜けるには、強引なやり方になるが、USBケーブルをいったん抜いて電源を切る。

(2)プログラムの実行

```bash

ampy put ESP32_starwars.py

picocom /dev/ttyUSB0 -b115200

import ESP32_starwars

```

実際に実行するプログラムは以下になる：  

以下の部分は自分のWiFi環境に合わせて変更すること：   
WIFI_SSID = "your_ssid"   
WIFI_PASSWD = "your_passwd"  

ESP32_starwars.py
```python

# NIC setup
import network, socket

WIFI_SSID = "your_ssid"
WIFI_PASSWD = "your_passwd"

def wlan_connect(ssid='SSID', password='PASSWD'):
    import network
    wlan = network.WLAN(network.STA_IF)
    if not wlan.active() or not wlan.isconnected():
        wlan.active(True)
        print('connecting to:', ssid)
        wlan.connect(ssid, password)
        while not wlan.isconnected():
            pass
  

wlan_connect(ssid=WIFI_SSID, password=WIFI_PASSWD)

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
printを使用している都合上、コンソールで実行する必要がある。途中でエラーになるようだがWiFi動作の確認には使える。

## ボードにプログラムを書き込む(自動起動になる)
書き込みたいプログラムをxxxx.pyとすると
以下のようにする：
```bash

ampy put xxxx.py /main.py 
# 実行したいプログラムをmain.pyとして書き込む
# ボードの[reset]を押して起動する

#以下で書き込んであるプログラムが確認できる
ampy ls
/boot.py
/main.py
```

## コンソールで実行したい場合
実行したいプログラムをESP32_starwars.pyとすると
以下のようにする：

```bash

ampy put ESP32_starwars.py
picocom /dev/ttyUSB0 -b115200

import ESP32_starwars

```


## 参考情報  
[ＥＳＰ３２－ＤｅｖＫｉｔＣ　ＥＳＰ－ＷＲＯＯＭ－３２開発ボード](http://akizukidenshi.com/catalog/g/gM-11819/)    
[MicroPython - Quick reference for the ESP32](https://docs.micropython.org/en/latest/esp32/quickref.html#)    
[Streaming Data from ESP32 using MicroPython and MQTT](https://github.com/gloveboxes/ESP32-MicroPython-BME280-MQTT-Sample/blob/master/Micropython.md)   
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)　　　

以上
