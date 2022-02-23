


2020/1/24

MicroPython DHT11 sensor 
# MicroPython DHT11 sensor 

## 概要
MicroPythonに温度/湿度センサーを接続する方法について説明する。
microbitのMicroPythonと他のMicroPythonで方法が異なるので、２つに分けて説明する。
ここでは、linux環境でのインストール方法について説明する。   

## 準備
(1)microbit-MicroPython用ツール(ufs)のインストール
```bash

pip install microfs
pip install --no-cache --upgrade micro
```

(2)他のMicroPython用ツール(ampy)のインストール   
AMPY_PORTは、自分の環境に合わせる。
```bash

pip install adafruit-ampy
export AMPY_PORT=/dev/ttyUSB0
export AMPY_BAUD=115200
```

(3)piccomのインストール
MicroPyhonのコンソールとして使用する。
```bash

sudo apt-get install piccom
```
以上のうち、exportしているものは、.bashrcに登録することを勧める。

## microbit MicroPythonの場合
microbit+Grove_Shieldのslot[P1/P15]にDHT11センサーを接続する。

```bash

# 内部モジュールの確認

picocom /dev/ttyACM0 -b115200

>>> help('modules')
__main__          love              os                time
antigravity       machine           radio             ucollections
array             math              random            ustruct
audio             microbit          speech            utime
builtins          micropython       struct
collections       music             sys
gc                neopixel          this
Plus any modules on the filesystem
>>> 


# DHT11モジュールのインストール
# microbitの場合、dhtが内蔵モジュールになっていないので
# 外部モジュールをインストールする
git clone https://github.com/rhubarbdog/microbit-dht11.git
cd microbit-dht11
ufs put dht11.py

cd ..

ufs dht11_test.py main.py
# 書き込みが終わったら[reset]を押してプログラムを起動する。

```
動作した実績があるので、ここに挙げたが   
原因が不明だが、かなりの確率でメモリが足りない旨のエラーがでて
実行できない場合がある。


テストプログラム   
dht_test.py
```python

from microbit import *
from dht11 import DHT11

DEGREES = u'\xb0'

sensor = DHT11(pin1) # slot[P1/P15] @Grove Shield

while True:
    t, h = sensor.read()
    display.scroll("{}C,{}%".format(t, h))
    print("%2.1f%sC  %2.1f%% " % (t, DEGREES, h))
    sleep(500)
    display.scroll("{}C,{}%".format(t, h))
    sleep(500)
```

## Nucleo_F446REのMicroPythonの場合
Nucleo_F446RE + Grove_Sheildのslot#7(D7)にDHT11センサーを接続する。

```bash

# 内蔵モジュールの確認

picocom /dev/ttyACM0 -b115200

>>> help('modules')
__main__          math              ucollections      urandom
_onewire          micropython       uctypes           ure
builtins          network           uerrno            uselect
cmath             onewire           uhashlib          usocket
dht               pyb               uheapq            ustruct
framebuf          stm               uio               utime
gc                sys               ujson             utimeq
lcd160cr          uarray            umachine          uzlib
lcd160cr_test     ubinascii         uos
Plus any modules on the filesystem
>>> 

# dhtモジュールが存在することの確認ができたので
# 内蔵モジュールdhtを使用する。

# RAM実行で動作確認を行なう
ampy run mp_dht11_test.py

# RAMでの動作確認が終わったらフラッシュに書き込む
ampy put mp_dht11_test.py /flash/main.py

# 書き込みが終わったら[reset]を押してプログラムを起動する

picocom /dev/ttyACM0 -b115200
#[reset]を押して再起動する

# 以下、出力ログ
15.0℃  34.0% 
15.0℃  34.0% 
15.0℃  34.0% 
15.0℃  34.0% 
15.0℃  34.0% 
15.0℃  34.0% 
15.0℃  34.0% 
15.0℃  33.0% 
15.0℃  34.0% 
15.0℃  34.0% 
15.0℃  34.0% 
15.0℃  34.0% 
Ctrl-C

```

テストプログラム   
mp_dht11_test.py
```python

# DHT11(blue)

import time

from dht import DHT11
import machine

while True:
   # Nucleo_F446RE + Grobe-Base slot#7(D7)
   d=DHT11(machine.Pin('A8')) # slot#7(D7)
   d.measure()
   temp=d.temperature()
   humi=d.humidity()
   print("%2.1f%s  %2.1f%% " % (temp, '℃',humi))
#   print('temp:'+str(temp),'humi:'+str(humi))
   time.sleep(1)  # sleep for 1 sec

```



## 参考情報  

[Nucleo-F446RE pin layout](https://os.mbed.com/platforms/ST-Nucleo-F446RE/)   
[Grove - 温度や湿度センサー](http://wiki.seeedstudio.com/jp/Grove-TemperatureAndHumidity_Sensor/)   
[Grove - DHT11](https://jp.seeedstudio.com/Grove-Temperature-Humidity-Sensor-DHT11.html)   
[micro:bit用GROVEシールド v2.0](https://www.switch-science.com/catalog/5434/)   
[Grove - Base Shield v1.2](https://seeeddoc.github.io/Grove-Base_Shield_v1.2/)  
[MicroFS](https://github.com/ntoll/microfs)  [ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)　　　

以上
