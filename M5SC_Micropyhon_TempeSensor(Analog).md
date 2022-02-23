
2020/2/3

M5StickC MicroPython Tempe Sensor(Analog)
# M5StickC MicroPython Tempe Sensor(Analog)

## 概要
M5StickCのGroveコネクタに温度センサー(アナログ)を接続する

## 参照URL

[ESP32/ESP8266 Analog Readings with MicroPython](https://randomnerdtutorials.com/esp32-esp8266-analog-readings-micropython/)    
[ESP32/ESP8266 PWM with MicroPython – Dim LED](https://randomnerdtutorials.com/esp32-esp8266-pwm-micropython/)   
[ESP32/ESP8266 Digital Inputs and Digital Outputs with MicroPython](https://randomnerdtutorials.com/esp32-esp8266-digital-inputs-digital-outputs-micropython/)   
[ESP32/WROOM32](https://i0.wp.com/randomnerdtutorials.com/wp-content/uploads/2018/08/esp32-pinout-chip-ESP-WROOM-32.png)   
[M5StickC非公式日本語リファレンス(pinoutあり)](https://lang-ship.com/reference/unofficial/M5StickC/)   
[Grove  Sensor  Grove - Rotary Angle Sensor](http://wiki.seeedstudio.com/Grove-Rotary_Angle_Sensor/)   
[Grove 温度センサ - Grove Temperature Sensor](https://jp.seeedstudio.com/Grove-Temperature-Sensor.html)   


## デモ・スクリプト
M5StickCのGroveコネクタに「Grove - 温度センサー」を接続し、LCDに温度を表示する。


M5SC_demo_TempeSensor.py
```python

# Tempe Sensor Demo
# for LCD display
from m5stack import *
from m5ui import *
from uiflow import *

setScreenColor(0x0)
label0 = M5TextBox(48, 1, " Demo Running...", lcd.FONT_Default,0xff0378, rotate=90)

# M5StickC use Grove Temp
from machine import Pin, ADC
from time import sleep
import math

# setup AnalogIn for Grove connector
g33 = ADC(Pin(33))
g33.atten(ADC.ATTN_11DB) # Full range: 3.3v
#g33.atten(ADC.ATTN_6DB)  # full range: 2.0V

def a2tempe(a):
   #B = 4275 # v1.2 (B value of the thermistor)
   B = 3975 # v1.0 (B value of the thermistor)
   R0 = 100000 #  R0 = 100k
   #
   # R = 4095.0/a-1.0
   #vcnv = 3145*5/3.3
   vcnv = 3145*5/3.0 # assume real voltage is 3.0V
   R = vcnv/a-1.0
   R = R0*R
   # convert to temperature via datasheet
   tempe  = 1.0/(math.log(R/R0)/B+1/298.15)-273.15
   return tempe

while True:
   ain = g33.read()
   tempe = a2tempe(ain)
   label0.setText(' Tempe:'+str(tempe))
   print(a2tempe(ain))
   sleep(0.1)

```
注意：  
・温度センサーはバージョンによってＢの値を変更する必要がある。  
・Groveのコネクタに来ている電圧は3.3Vにはなっていないようなので、3.0Vと想定したプログラムになっている。
だいたい確からしい温度が表示されていると思う。

## 実行

```bash

ampy run M5SC_demo_TempeSensor.py 

```
原因は、まだ不明だが、ときどき/dev/ttyUSB0が出現しなくなったり、出現していても
ampyで正常動作しない場合がある。そのときは、Cloudにプログラムをコピー&ペーストして
実行するほうが確実のようだ。Cloudに接続後に/dev/ttyUSB0が復活したりするので
なにかしら初期化の問題かもしれない。

以上
 