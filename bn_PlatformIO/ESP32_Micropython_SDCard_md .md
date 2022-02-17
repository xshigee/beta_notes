
2020/2/12

ESP32 MicroPython SDCard
# ESP32 MicroPython SDCard

## 概要
ESP32のMicroPythonでSDCardを使う方法について述べる。ここでは、linux環境でのインストール方法について説明する。   

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

## ESP32とSDCardスロットの接続
ブレッドボードにESP32とSDCardスロットを取り付け、配線する。   
SDCardスロット(AE-MICRO-SD-DIP)は以下のものを使用した：  
[マイクロＳＤカードスロットＤＩＰ化キット[AE-MICRO-SD-DIP]](http://akizukidenshi.com/catalog/g/gK-05488/)   
[microSDカードスロットモジュール取扱説明書](http://akizukidenshi.com/download/ds/akizuki/K-5488_AE-MICRO-SD-DIP.pdf)  

配線表:  
| ピン名称 | AE-MICRO-SD-DIP | ESP32 |  
| :-----: | :-------------: | :---: |  
| SCL | (5)CLK | IO14 |  
| MISO	| (7)DAT0    |IO12 |  
| MOSI	| (3)CMD  |   IO13 |  
| CS | (2)CD/DAT3 | IO15 |  
| 3.3V | (4)VDD | 3V3 |  
| GND	| (6)VSS | GND|  


特にプルアップ抵抗などは使用せず、直接、ピンとピンをジャンパー線で繋いだ。

使用するSDはFATでフォーマットしたものにテスト用になにかしらのファイルを入れておく。

## 動作確認用スクリプト

「picocom /dev/ttyUSB0 -b115200」
でREPLに入り以下を実行する：

```python

from machine import Pin, SDCard
import os
sd = SDCard(slot=3, width=1, sck=Pin(14), mosi=Pin(13), miso=Pin(12),cs=Pin(15))　# slot#3を使用する
os.mount(sd, '/sd')

os.chdir('/sd')
os.listdir()
出力例：
['lib', 'ESP32_OSCrecvBundled_test.py', 'ESP32_demo_neopixel.py', 'ESP32_starwars.py', 'M5S_OSCrecvBundled_test.py', 'TEST.txt', 'sdmain.py', 'wlan.py']
# この時点でエラーがなく、SDのファイルが表示されたら動作確認としてはＯＫとなる

import sys
sys.path
出力例：
['', '/lib']

sys.path[1]='/sd/lib'　# ライブラリのパスをSDに切り替える
import ESP32_OSCrecvBundled_test # SDに置いてあるプログラム(ESP32_OSCrecvBundled_test.py)をロードして実行する

```

## 参考情報  

[SDcard/ハードウェア固有の詳細/ESP32](https://micropython-docs-ja.readthedocs.io/ja/latest/library/machine.SDCard.html) 

[ＥＳＰ３２－ＤｅｖＫｉｔＣ　ＥＳＰ－ＷＲＯＯＭ－３２開発ボード](http://akizukidenshi.com/catalog/g/gM-11819/)    
[MicroPython - Quick reference for the ESP32](https://docs.micropython.org/en/latest/esp32/quickref.html#)    
[Streaming Data from ESP32 using MicroPython and MQTT](https://github.com/gloveboxes/ESP32-MicroPython-BME280-MQTT-Sample/blob/master/Micropython.md)   
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)　　　

以上
