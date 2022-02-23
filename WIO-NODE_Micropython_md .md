
2020/2/19

WIO-NODE MicroPython Install
# WIO-NODE MicroPython Install

## 概要
WIO-NODE(ESP8266)にMicroPythonのfirewareを書き込む方法についてまとめた。
ここでは、linux環境でのインストール方法について説明する。   

## 準備
ツールをインストール前に環境整備として以下を設定する：  
(1)esptool.pyのインストール   
以下、参照のこと：  
[ESP8266 and ESP32 serial bootloader utility](https://github.com/espressif/esptool)

(2)ampyのインストール   
AMPY_PORTは、自分の環境に合わせる。
```bash

pip install adafruit-ampy
export AMPY_PORT=/dev/ttyUSB0
export AMPY_BAUD=115200
```

(3)piccomのインストール
```bash

sudo apt-get install piccom
```
以上のうち、exportしているものは、.bashrcに登録することを勧める。

## WIO-NODEとUSBシリアルを接続する
http://wiki.seeedstudio.com/Grove_System/#grove-uart

以下の表のようにWIO-NODEのシリアル用GroveコネクタにUSBシリアルを接続する。
| pin |	WIO-NODE | USB-serial |
| :--: | :---: | :----: |
| pin1(YELLOW) | RX |	TXD |
| pin2(WHITE) | TX |	RXD |
| pin3(RED) |	VCC |	NC |
| pin4(BLACK) |	GND |	GND |

NC: None Connection

注意：   
YELLOW/WHITEが入れ替わっているケーブルもあるので注意のこと。(M5Stackのものが入れ替わっているようだ)

## firewareのダウンロード＆書き込み
```bash

wget http://micropython.org/resources/firmware/esp8266-20191220-v1.12.bin

# WIO-NODEを書き込みモードにするために
# （「Func」「RST」の２つのタクトスイッチがあるので）
# [Func]を押したまま、[RST]を押して離す.
# （その後、[Func]を離す）

esptool.py --port /dev/ttyUSB0 erase_flash
esptool.py --port /dev/ttyUSB0 --baud 460800 write_flash --flash_size=detect 0 esp8266-20191220-v1.12.bin

# 書き込みが完了したら、いったん、ボードの電源を抜き差しする。

```

## REPLログ
```bash


picocom /dev/ttyUSB0 -b115200

MPY: soft reboot
MicroPython v1.12 on 2019-12-20; ESP module with ESP8266
Type "help()" for more information.

>>> import os
>>> os.uname()
(sysname='esp8266', nodename='esp8266', release='2.2.0-dev(9422289)', version='v1.12 on 2019-12-20', machine='ESP module with ESP8266')

>>> import gc
>>> gc.collect()
>>> gc.mem_free()
32928

>>> help('modules')
__main__          http_client_ssl   uasyncio/__init__ upysh
_boot             http_server       uasyncio/core     urandom
_onewire          http_server_ssl   ubinascii         ure
_webrepl          inisetup          ucollections      urequests
apa102            lwip              ucryptolib        urllib/urequest
btree             machine           uctypes           uselect
builtins          math              uerrno            usocket
dht               micropython       uhashlib          ussl
ds18x20           neopixel          uheapq            ustruct
esp               network           uio               utime
example_pub_button                  ntptime           ujson             utimeq
example_sub_led   onewire           umqtt/robust      uwebsocket
flashbdev         port_diag         umqtt/simple      uzlib
framebuf          ssd1306           uos               webrepl
gc                sys               upip              webrepl_setup
http_client       uarray            upip_utarfile     websocket_helper
Plus any modules on the filesystem
>>> 


```

## WebREPLを使う
```bash
>>> 
>>> import webrepl_setup
WebREPL daemon auto-start status: disabled

Would you like to (E)nable or (D)isable it running on boot?
(Empty line to quit)
> E
To enable WebREPL, you must set password for it
New password (4-9 chars): your_passwd(任意のパスワードを設定する)
Confirm password: your_passed
Changes will be activated after reboot
Would you like to reboot now? (y/n) (yでリブートする) 

WebREPL daemon started on ws://0.0.0.0:8266
Started webrepl in normal mode

# (1)WIO-NODEがWiFi-APとして起動する。ホストＰＣからはMicroPython-xxxxxxというSSIDが見える。
# (2)ホストPCのブラウザからhttp://micropython.org/webrepl/にアクセスする。（後で接続を切るので、その内容をキャッシュする）
# (3)ホストＰＣでMicroPython-xxxxxxへ接続する。WiFiのパスワードは「micropythoN」とする
# (4)キャシュしたWebページのConnectボタンをクリックし、先ほど設定したパスワード(your_passwd)を入力してREPLに接続する。
#   この時点でシリアルと接続したと同じ感覚でweb経由でREPLが使用できる。
#  (ただし、アクセスポイントとしてESP8266に接続しているので、外部のネットワークのアクセスは不可になっている)

# 次回からWebREPLを使用するときは、上の(1)から(4)の手順を行なうだけで良いので、シリアル接続は不要となる。

```

## Groveコネクタ使用時の注意
WIO-NODEでは Groveコネクタにはデフォルトでは 3.3V電源が供給されていない。
したがって、Groveコネクタを使用する際は、以下のスクリプトを実行して電源をオンにする必要がある。
```python

import machine
p15 = machine.Pin(15, machine.Pin.OUT)
p15.value(1)    # Enable power 3V3B
```

## 参考情報  

[Wio Node](https://www.switch-science.com/catalog/2799/)  
[ESP-WROOM-02 Arduino互換ボード](https://www.switch-science.com/catalog/2620/)   
[lvidarte/esp8266](https://github.com/lvidarte/esp8266/wiki/MicroPython:-Examples)   
[MicroPython tutorial for ESP826](https://docs.micropython.org/en/v1.9.2/esp8266/esp8266/tutorial/index.html)   
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)  

[ESP8266(WIO NODE)でMicroPythonを動かしてみた](https://blog.boochow.com/article/esp8266wio-node-micropython.html)   
[話題のWioNodeを買うも、MicroPython載せてIFTTTするの巻](https://flogics.com/wp/ja/2017/12/wio-node-micropython-ifttt/)  

以上
