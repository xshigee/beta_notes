
2020/3/6

ESP32/ESP8266 MicroPython Clock
# ESP32/ESP8266 MicroPython Clock

## 概要
ESP32/ESP8266のMicroPythonでRTCを現在時刻に合わせる方法を記する。ここでは、デモとして簡単な時計表示を行なう。

## micropythonスクリプト

ESP32_8266_clock.py  
以下は自分の環境に合わせる：  
ssid = 'your_ssid'  
password = 'your_passwd'    

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
# ESP32_8266_clock.py

from machine import Pin, RTC
import network, urequests, utime
import ujson
rtc = RTC()

url_jst = "http://worldtimeapi.org/api/timezone/Asia/Tokyo" # see http://worldtimeapi.org/timezones

retry_delay = 5000 # interval time of retry after a failed Web query
response = urequests.get(url_jst)

# parse JSON
parsed = response.json()
#parsed=ujson.loads(res) # in case string
datetime_str = str(parsed["datetime"])
year = int(datetime_str[0:4])
month = int(datetime_str[5:7])
day = int(datetime_str[8:10])
hour = int(datetime_str[11:13])
minute = int(datetime_str[14:16])
second = int(datetime_str[17:19])
subsecond = int(round(int(datetime_str[20:26]) / 10000))
        
# update internal RTC
rtc.datetime((year, month, day, 0, hour, minute, second, subsecond))

# setup day of the week
daysOfTheWeek = "Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"
tm = utime.localtime(utime.mktime(utime.localtime()))

while True:
  # generate formated date/time strings from internal RTC
  date_str = "Date: {0:4d}/{1:02d}/{2:02d}".format(*rtc.datetime())+' ('+daysOfTheWeek[tm[6]]+')'
  time_str = "Time: {4:02d}:{5:02d}:{6:02d}".format(*rtc.datetime())
  print(date_str)
  print(time_str)
  print('-----------')
  utime.sleep(1)

#=============================================
```
ここでは、[World Time API](http://worldtimeapi.org/timezones)のサーバーから現在時刻を取得して時刻合わせを行なう。

## 他の方法

ネットワークが接続済みが前提で以下のスクリプトでも時刻合わせができる。(NTPサーバーを利用している)

```python
#===========================================
from machine import RTC
import utime
import ntptime
rtc = RTC()
ntptime.settime() # setup time by remote NTP server

def get_jst():
   tm = utime.localtime(utime.time()) # UTC now
   jst = str(tm[0])+'/'+str(tm[1])+'/'+str(tm[2])+' '+str((tm[3]+9)%24)+':'+str(tm[4])+':'+str(tm[5])
   return jst

#===========================================
```


## 実行画面

```python

picocom /dev/ttyUSB0 -b115200

import ESP32_8266_clock

Date: 2020/03/06 (Fri)
Time: 12:29:20
-----------
Date: 2020/03/06 (Fri)
Time: 12:29:21
-----------
Date: 2020/03/06 (Fri)
Time: 12:29:22
-----------
Date: 2020/03/06 (Fri)
Time: 12:29:23
-----------
Date: 2020/03/06 (Fri)
Time: 12:29:24
-----------
Date: 2020/03/06 (Fri)
Time: 12:29:25
-----------
Date: 2020/03/06 (Fri)
Time: 12:29:26
-----------
...
```

## 参考情報  

[World Time API](http://worldtimeapi.org/timezones)  

[Very Simple MicroPython ESP8266/ESP-12 Web Clock](https://www.hackster.io/alankrantas/very-simple-micropython-esp8266-esp-12-web-clock-3c5c6f#things)  

以上
