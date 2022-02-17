
2020/2/3

M5StickC MicroPython AnalogIn_PWM Demo
# M5StickC MicroPython AnalogIn_PWM Demo

## 概要
M5StickCでのAnalogInとPWMのデモについて記する。

## 参照URL

[ESP32/ESP8266 Analog Readings with MicroPython](https://randomnerdtutorials.com/esp32-esp8266-analog-readings-micropython/)    
[ESP32/ESP8266 PWM with MicroPython – Dim LED](https://randomnerdtutorials.com/esp32-esp8266-pwm-micropython/)   
[ESP32/ESP8266 Digital Inputs and Digital Outputs with MicroPython](https://randomnerdtutorials.com/esp32-esp8266-digital-inputs-digital-outputs-micropython/)   
[ESP32/WROOM32](https://i0.wp.com/randomnerdtutorials.com/wp-content/uploads/2018/08/esp32-pinout-chip-ESP-WROOM-32.png)   
[M5StickC非公式日本語リファレンス(pinoutあり)](https://lang-ship.com/reference/unofficial/M5StickC/)   
[Grove  Sensor  Grove - Rotary Angle Sensor](http://wiki.seeedstudio.com/Grove-Rotary_Angle_Sensor/)   


## デモ・スクリプト
PWMには内蔵LEDを利用して、AnalogInは、Groveコネクタに「Grove - Rotary Angle Sensor」を接続する。
実行すると、Angle-Sensorの回転角度に対応してLEDの明るさが変化する。

M5SC_demo_AinPWM.py
```python

# for LCD display
from m5stack import *
from m5ui import *
from uiflow import *

setScreenColor(0x0)
label0 = M5TextBox(48, 1, " Demo Running...", lcd.FONT_Default,0xff0378, rotate=90)

# M5StickC use bultin LED and Grove Vol
from machine import Pin, ADC, PWM
from time import sleep

AnalogMax=1051

# setup LED PWM
frequency = 5000
g10 = PWM(Pin(10), frequency)

# setup AnalogIn for Grove connector
g33 = ADC(Pin(33))

while True:
  ain = 100-(g33.read()/AnalogMax)*100
  print(ain)
  g10.duty(ain)
  sleep(0.01)

```


## 実行

```bash

ampy run M5SC_demo_AinPWM.py 

```
原因は、まだ不明だが、ときどき/dev/ttyUSB0が出現しなくなったり、出現していても
ampyで正常動作しない場合がある。そのときは、Cloudにプログラムをコピー&ペーストして
実行するほうが確実のようだ。Cloudに接続後に/dev/ttyUSB0が復活したりするので
なにかしら初期化の問題かもしれない。

以上

