
2020/2/2

MicroPython AnalogInのボードによる仕様差分
# MicroPython AnalogInのボードによる仕様差分

## 概要
MicroPythonのAnalogInのボードによる仕様差分について、プログラムを示す形で明確化する。

## 参照URL

[ESP32/ESP8266 Analog Readings with MicroPython](https://randomnerdtutorials.com/esp32-esp8266-analog-readings-micropython/)    
[ESP32/ESP8266 PWM with MicroPython – Dim LED](https://randomnerdtutorials.com/esp32-esp8266-pwm-micropython/)   
[ESP32/ESP8266 Digital Inputs and Digital Outputs with MicroPython](https://randomnerdtutorials.com/esp32-esp8266-digital-inputs-digital-outputs-micropython/)   
[ESP32/WROOM32](https://i0.wp.com/randomnerdtutorials.com/wp-content/uploads/2018/08/esp32-pinout-chip-ESP-WROOM-32.png)   
[ESP8266/ESP-WROOM-02開発ボード](https://www.switch-science.com/catalog/2620/)  


## Nucleo_F446RE

```python

from machine import Pin, ADC
from time import sleep

# use A1 for analog input (Nucleo_F446RE)
a1 = ADC(machine.Pin('A1'))

while True:
  print(a1.read())
  sleep(0.1)

```

以下のようにREPLで実行してみるとA0が使用不可なことが分かる：
```bash

>>> a0= ADC(machine.Pin('A0'))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: pin A0 does not have ADC capabilities
>>> a1= ADC(machine.Pin('A1'))
>>> a2= ADC(machine.Pin('A2'))
>>> a3= ADC(machine.Pin('A3'))
>>> a4= ADC(machine.Pin('A4'))
>>> a5= ADC(machine.Pin('A5'))
>>> machine.Pin('A3')
Pin(Pin.cpu.B0, mode=Pin.ANALOG)
>>> machine.Pin('A0')
Pin(Pin.cpu.A0, mode=Pin.IN) # A0 は　mode=Pin.ANALOG　になっていない。

# ボリューム抵抗を最大にすると12ビットであることが分かる
>>> a2= ADC(machine.Pin('A2'))
>>> a2.read()
4095
>>> 
```


## ESP8266
AnalogInは一つしか無い(A0,0-1023)
```python

from machine import Pin, ADC
from time import sleep

a0 = ADC(0)

while True:
  print(a0.read())
  sleep(0.1)


```


## ESP32
以下のピンがAnalogInとして使用できる:   
0, 2, 4, 12, 13, 14, 15, 25, 26, 27, 32, 33, 34, 35, 36, 39   
レンジは、0-4095   


```python

from machine import Pin, ADC
from time import sleep

p34 = ADC(Pin(34))

while True:
  print(p34.read())
  sleep(0.1)

```


以上
