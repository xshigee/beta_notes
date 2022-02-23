
2020/2/5

ESP8266/Nucleo-F446RE MicroPython DHT11　demo
# ESP8266/Nucleo-F446RE MicroPython DHT11　demo

## 概要
MicroPythonボードのGroveコネクタに温度・湿度センサー(DHT11)を接続する。

## 参照URL

[ESP32/ESP8266 Analog Readings with MicroPython](https://randomnerdtutorials.com/esp32-esp8266-analog-readings-micropython/)    
[ESP32/ESP8266 PWM with MicroPython – Dim LED](https://randomnerdtutorials.com/esp32-esp8266-pwm-micropython/)   
[ESP32/ESP8266 Digital Inputs and Digital Outputs with MicroPython](https://randomnerdtutorials.com/esp32-esp8266-digital-inputs-digital-outputs-micropython/)   
[ESP32/WROOM32](https://i0.wp.com/randomnerdtutorials.com/wp-content/uploads/2018/08/esp32-pinout-chip-ESP-WROOM-32.png)   
  
[ESP-WROOM-02 Arduino互換ボード](https://www.switch-science.com/catalog/2620/)   
[NUCLEO F446RE MicroPython インストール方法](https://beta-notes.way-nifty.com/blog/2020/01/post-459022.html)   
[STM32F4-Discovery」にMicroPythonをインストールする](https://beta-notes.way-nifty.com/blog/2020/02/post-6822aa.html)  

[Grove - 温度および湿度センサー (DHT11)](https://jp.seeedstudio.com/Grove-Temperature-Humidity-Sensor-DHT11.html)  
[Grove 温度および湿度センサPro – DHT22 / AM2302 - Grove Temperature & Humidity Sensor Pro](https://jp.seeedstudio.com/Grove-Temperature-Humidity-Sensor-Pro-AM2302.html)   
  
[Grove - LCD RGB Backlight](https://www.seeedstudio.com/Grove-LCD-RGB-Backlight.html)   
[Grove  Sensor  Grove - Rotary Angle Sensor](http://wiki.seeedstudio.com/Grove-Rotary_Angle_Sensor/)   
[Grove 温度センサ - Grove Temperature Sensor](https://jp.seeedstudio.com/Grove-Temperature-Sensor.html)   

## 関連モジュールのインストール
内蔵モジュールdhtを使用するので、インストールするものはない。

## デモ・スクリプト
Nucleo-F446RE + Grove-Base-Sheild のＤ４に「Grove - 温度および湿度センサー (DHT11)」を接続する。または、
「ESP-WROOM-02 Arduino互換ボード(ESP8266)」 +  Grove-Base-Sheild のD7に「Grove - 温度および湿度センサー (DHT11)」を接続する。   
以下に２つのボードのスクリプトを挙げたが、DHT11の初期化の１行以外は同じである。
ESP8266のほうは使用可能なピン番号が決まっているようで、ちがうピン番号ではエラーになった。

STM32_demo_DHT11.py
```python

# F446RE
from dht import DHT11
import machine
from time import sleep

d = DHT11(machine.Pin('D4'))
while True:
   d.measure()
   tempe=d.temperature()
   humi=d.humidity()
   print('tempe:'+str(tempe)+ ' humi:'+str(humi))
   sleep(1)

```

ESP8266_demo_DHT11.py
```python

# ESP8266
from dht import DHT11
import machine
from time import sleep

d = DHT11(machine.Pin(14)) # D7 (Arduino Pin)
while True:
   d.measure()
   tempe=d.temperature()
   humi=d.humidity()
   print('tempe:'+str(tempe)+ ' humi:'+str(humi))
   sleep(1)

```
なお、DHT11をDHT12に置き換えれば、DHT12センサーでも動作するはずである。


## 実行

```bash

# flashに書き込む
ampy put xxxx_demo_DHT11.py

# picocomなどでシリアルのコンソールに入る
>>>
>>>
# 以下で実行する
import xxxx_demo_DHT11


```


以上
 
