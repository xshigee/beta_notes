
2020/3/14

Nucleo MicroPython Grove-RTC(i2c)
# Nucleo MicroPython Grove-RTC(i2c)
## 概要
Nucleo-MicroPythonにGrove-RTC(i2c)を接続する。
ここでは、linux環境でのインストール方法について説明する。   

## 配線
Grove-Base-Shield経由でGrove-RTCを接続する。

## 関連モジュールのインストール
以下の手順でインストールする：

Module Install:
```bash

git clone https://github.com/mcauser/micropython-tinyrtc-i2c.git
cd micropython-tinyrtc-i2c.git
ampy put ds1307.py 

```

## 動作確認用スクリプト


Test Script
```python
# I2C-VCC needs 5V for this device
from machine import Pin, I2C
i2c = I2C(1)
i2c.scan()
import ds1307
ds = ds1307.DS1307(i2c)
# enable oscillator
ds.halt(False)
# set the datetime 2020/3/14 10:05:21 Sat(6)
ds.datetime((2020, 3, 14, 6, 10, 05, 21, 0))
# read datetime
ds.datetime()

# いったん、ボードの電源をオフする
from machine import Pin, I2C
i2c = I2C(1)
import ds1307
ds = ds1307.DS1307(i2c)
# read datetime
ds.datetime()
# ここで正しい時刻を表示すれば
# バックアップ電池が動作していることになる

# simple clock
from time import sleep
# setup day of the week
daysOfTheWeek = "", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"

while True:
  # generate formated date/time strings from RTC
  date_str = 'Date: {0:4d}/{1:02d}/{2:02d}'.format(*ds.datetime())+' ('+daysOfTheWeek[ds.datetime()[3]]+')'
  time_str = 'Time: {4:02d}:{5:02d}:{6:02d}'.format(*ds.datetime())
  print(date_str)
  print(time_str)
  print('==============')
  sleep(1)

#-----------------------

```

## 参考情報  

[MicroPython TinyRTC I2C module](https://github.com/mcauser/micropython-tinyrtc-i2c)  

[GROVE - 高精度RTC](https://www.switch-science.com/catalog/3134/)  

[DS1307 datasheet](https://raw.githubusercontent.com/SeeedDocument/Grove-RTC/master/res/DS1307.pdf)  
[Grove-RTC Schematic](https://github.com/SeeedDocument/Grove-RTC/raw/master/res/Grove%20-%20RTC%20v1.1%20Sch.pdf)  
[wiki/Grove-RTC](http://wiki.seeedstudio.com/Grove-RTC/)


以上
