
2020/3/3

ESP32/ESP8266 MicroPython MQTT test
# ESP32/ESP8266 MicroPython MQTT test

## 概要
ESP32/ESP8266のMicroPythonで以下のリンクを参照してMQTTを使ってみた。
ここでは、linux環境でのインストール方法について説明する。   

[MicroPython – Getting Started with MQTT on ESP32/ESP8266](https://randomnerdtutorials.com/micropython-mqtt-esp32-esp8266/)  


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

## 関連モジュールのインストール
```bash

wget https://raw.githubusercontent.com/RuiSantosdotme/ESP-MicroPython/master/code/MQTT/umqttsimple.py
ampy umqttsimple.py

```
MQTT通信のテストで使用するので、2つのESP32/ESP8266に上のモジュールをインストールする。
2つのESP32/ESP8266を便宜上、ESP\#1,ESP\#2とする。

## ESP\#1側のスクリプト
MQTT01_test.py   
以下の部分は自分の環境に合わせる：  
ssid = 'your_ssid'  
password = 'your_passws'  
```python

# boot.py
ssid = 'your_ssid'
password = 'your_passws'
mqtt_server = '5.196.95.208'

# Complete project details at https://RandomNerdTutorials.com

import time
from umqttsimple import MQTTClient
import ubinascii
import machine
import micropython
import network
import esp
esp.osdebug(None)
import gc
gc.collect()

client_id = ubinascii.hexlify(machine.unique_id())
# ESP#1
topic_sub = b'notification'
topic_pub = b'hello'
# ESP#2
#topic_sub = b'hello'
#topic_pub = b'notification'

last_message = 0
message_interval = 5
counter = 0

station = network.WLAN(network.STA_IF)

station.active(True)
station.connect(ssid, password)

while station.isconnected() == False:
  pass

print('Connection successful')
print(station.ifconfig())
#===========================================
# ESP#1
# main.py

# Complete project details at https://RandomNerdTutorials.com

def sub_cb(topic, msg):
  print((topic, msg))
  if topic == b'notification' and msg == b'received':
    print('ESP received hello message')

def connect_and_subscribe():
  global client_id, mqtt_server, topic_sub
  client = MQTTClient(client_id, mqtt_server)
  client.set_callback(sub_cb)
  client.connect()
  client.subscribe(topic_sub)
  print('Connected to %s MQTT broker, subscribed to %s topic' % (mqtt_server, topic_sub))
  return client

def restart_and_reconnect():
  print('Failed to connect to MQTT broker. Reconnecting...')
  time.sleep(10)
  machine.reset()

try:
  client = connect_and_subscribe()
except OSError as e:
  restart_and_reconnect()

while True:
  try:
    client.check_msg()
    if (time.time() - last_message) > message_interval:
      msg = b'Hello #%d' % counter
      client.publish(topic_pub, msg)
      last_message = time.time()
      counter += 1
  except OSError as e:
    restart_and_reconnect()

```

## ESP\#2側のスクリプト
MQTT02_test.py   
以下の部分は、自分の環境に合わせる：  
ssid = 'your_ssid'  
password = 'your_passwd'  

```python
# boot.py
ssid = 'your_ssid'
password = 'your_passwd'
mqtt_server = '5.196.95.208'

# Complete project details at https://RandomNerdTutorials.com

import time
from umqttsimple import MQTTClient
import ubinascii
import machine
import micropython
import network
import esp
esp.osdebug(None)
import gc
gc.collect()

client_id = ubinascii.hexlify(machine.unique_id())
# ESP#1
#topic_sub = b'notification'
#topic_pub = b'hello'
# ESP#2
topic_sub = b'hello'
topic_pub = b'notification'

last_message = 0
message_interval = 5
counter = 0

station = network.WLAN(network.STA_IF)

station.active(True)
station.connect(ssid, password)

while station.isconnected() == False:
  pass

print('Connection successful')
print(station.ifconfig())
#===========================================
# ESP#2
# main.py

# Complete project details at https://RandomNerdTutorials.com

def sub_cb(topic, msg):
  print((topic, msg))

def connect_and_subscribe():
  global client_id, mqtt_server, topic_sub
  client = MQTTClient(client_id, mqtt_server)
  client.set_callback(sub_cb)
  client.connect()
  client.subscribe(topic_sub)
  print('Connected to %s MQTT broker, subscribed to %s topic' % (mqtt_server, topic_sub))
  return client

def restart_and_reconnect():
  print('Failed to connect to MQTT broker. Reconnecting...')
  time.sleep(10)
  machine.reset()

try:
  client = connect_and_subscribe()
except OSError as e:
  restart_and_reconnect()

while True:
  try:
    new_message = client.check_msg()
    if new_message != 'None':
      client.publish(topic_pub, b'received')
    time.sleep(1)
  except OSError as e:
    restart_and_reconnect()


```

## 実行/出力ログ
ESP\#1側：
```bash
>>> 
>>> import MQTT01_test
Connection successful
('192.168.0.4', '255.255.255.0', '192.168.0.1', '192.168.0.1')
Connected to 5.196.95.208 MQTT broker, subscribed to b'notification' topic
(b'notification', b'received')
ESP received hello message
(b'notification', b'received')
ESP received hello message
(b'notification', b'received')
ESP received hello message
(b'notification', b'received')
ESP received hello message
(b'notification', b'received')
ESP received hello message
...
...

```

ESP\#2側：
```bash

>>> import MQTT02_test
Connection successful
('192.168.0.31', '255.255.255.0', '192.168.0.1', '192.168.0.1')
Connected to 5.196.95.208 MQTT broker, subscribed to b'hello' topic
(b'hello', b'Hello #1')
(b'hello', b'Hello #2')
(b'hello', b'Hello #3')
(b'hello', b'Hello #4')
(b'hello', b'Hello #5')
(b'hello', b'Hello #6')
mqtt reconnect
(b'hello', b'Hello #7')
mqtt reconnect ok
(b'hello', b'Hello #8')
(b'hello', b'Hello #9')
(b'hello', b'Hello #10')
...
...


```

## Brokerサーバーについて
２つのESP間を仲介するBrokerサーバーについては公開されているtest.mosquitto.orgを使用した。
そのIPアドレスはpingで取得した。(pingの結果より、5.196.95.208となる)
```bash

$ ping test.mosquitto.org
PING test.mosquitto.org (5.196.95.208) 56(84) bytes of data.
64 bytes from test.mosquitto.org (5.196.95.208): icmp_seq=1 ttl=51 time=257 ms
...

```

## 注意事項
２つのESPを使用するので、同じホストPCに接続した場合、シリアルのデバイス名が、/dev/ttyUSB0,/dev/ttyUSB1となる。習慣的に/dev/ttyUSB0だけと思っていると動作しないことになるので、注意のこと。


## 参考情報  

[MicroPython – Getting Started with MQTT on ESP32/ESP8266](https://randomnerdtutorials.com/micropython-mqtt-esp32-esp8266/)  

[MQTT server - test.mosquitto.org](https://test.mosquitto.org/)  
[MQTT is a machine-to-machine (M2M)/"Internet of Things" ](http://mqtt.org/)  

[ＥＳＰ３２－ＤｅｖＫｉｔＣ　ＥＳＰ－ＷＲＯＯＭ－３２開発ボード](http://akizukidenshi.com/catalog/g/gM-11819/)    
[MicroPython - Quick reference for the ESP32](https://docs.micropython.org/en/latest/esp32/quickref.html#)    
[Streaming Data from ESP32 using MicroPython and MQTT](https://github.com/gloveboxes/ESP32-MicroPython-BME280-MQTT-Sample/blob/master/Micropython.md)   
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)　　　

以上
