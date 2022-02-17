
2020/1/29++

MaixPy(MicroPython) Blink
# MaixPy(MicroPython) Blink

## 概要
M5stickV/Maixduinoに内蔵されているMicroPython(MaixPy)でLEDを点滅する方法について記する。
(ホストPCとしてはubuntuを想定している)

## 事前準備
(1)ampyのインストール   

```bash

pip install adafruit-ampy
export AMPY_PORT=/dev/ttyUSB0
export AMPY_BAUD=115200
export AMPY_DELAY=0.5
```
重要：   
・他のMicroPythonボードと異なり「export AMPY_DELAY=0.5」が必要である。   
・シリアルポートが他のボートと異なり/dev/ttyACM0でなく/dev/ttyUSB0であることに注意。  

(2)picocomのインストール
```bash

sudo apt-get install picocom
```
以上のうち、exportしているものは、.bashrcに登録することを勧める。

(3)microSD(M5StickVの場合)    
M5StickVを使う場合、スクリプトが空のboot.pyの入ったmicroSD(FAT32)を用意して、起動前に挿入しておくこと。
そうすることで、起動時に内蔵flashにあるboot.pyではなくmicroSDからのboot.pyからの起動になる。
(空スクリプトのboot.pyを実行するので結果として何も実行していない状態になる)

(4)Grove-LED(M5StickVの場合)   
M5StickVのGroveコネクタにGrove-LEDを接続する。
Groveコネクタには、34,35のpinが外部に出ている。

## 実行プログラム

M5StickVの場合  
M5SV_blink.py
```python

import utime
from Maix import GPIO
from fpioa_manager import fm

fm.register(34, fm.fpioa.GPIOHS0)
p34=GPIO(GPIO.GPIOHS0,GPIO.OUT)

while True:
   p34.value(0)
   utime.sleep_ms(100)
   p34.value(1)
   utime.sleep_ms(100)

```

maixduinoの場合    
MXD_blink.py
```python

import utime
from Maix import GPIO
from fpioa_manager import fm

fm.register(5,fm.fpioa.GPIO0)
led_r=GPIO(GPIO.GPIO0,GPIO.OUT)

while True:
   led_r.value(0)
   utime.sleep_ms(100)
   led_r.value(1)
   utime.sleep_ms(100)

```

## プログラム実行(RAM内)
```bash

$ ampy run xxxx_blink.py 

Maixduinoの場合、シリアルポートのLED(TX-K201)を利用しているので、実行後、ホストと通信できない状態になる。
通信再開したい場合、ボードとホストＰＣをつないでいるUSBを抜き刺しする。

```

## Maixduinoのピン配置(pinout)
ネットにある配線図から以下の表を作った：   
MaixPyで利用するピン番号は、左側のK210モジュールのピン番号になる。   
(例）   
本件のプログラムではシリアルについているLED(TX-K201)を利用したが、プログラムに使用するピン番号は
D1(TX)ではなく、5(IO05)になる。
ちなみに、プログラムでboard_info.ISP_TXとすると5を返す。


### 1. Digital I/O (1)
| K210 Pin Name | Arduino Pin Name |  
| :---: | :---: |  
| IO30 | SCL |
| IO31 | SDA |
| IO03 | D13 |
| IO10 | D12 |
| IO11 | D11 |
| IO12 | D10 |
| IO13 | D09 |
| IO14 | D08 |
### 2. Digital I/O (2)
| K210 Pin Name | Arduino Pin Name |
| :---: | :---: |
| IO15 | D07 |
| IO32 | D06 |
| IO24 | D05 |
| IO23 | D04 |
| IO22 | D03 |
| IO21 | D02 |
| IO05 | D01(TX) |
| IO04 | D00(RX) |

### 3. Analog In
| ESP32 Pin Name | Arduino Pin Name |
| :---: | :---: |
| ESP32_33 | A05 |
| ESP32_32 | A04 |
| ESP32_35 | A03 |
| ESP32_34 | A02 |
| ESP32_39 | A01 |
| ESP32_36 | A00 |
(ADCはESP32のものを使っている)

## Maixduinoのボード情報
REPLで以下のようにボード情報を知ることができる：
```python

>>> from board import board_info
# 以下、ArduinoのD0-D13ピンに対応したK210のピン番号が表示される
#(上の表の情報と同じ)
>>> board_info.D
[4, 5, 21, 22, 23, 24, 32, 15, 14, 13, 12, 11, 10, 3]

>>>
>>> board_info.pin_map()
|---Pin----|-----Function----------|
|   0      |     JTAG_TCK          |
+----------|-----------------------+
|   1      |     JTAG_TDI          |
+----------|-----------------------+
|   2      |     JTAG_TMS          |
+----------|-----------------------+
|   3      |     JTAG_TDO          |
+----------|-----------------------+
|   4      |     ISP_RX            |
+----------|-----------------------+
|   5      |     ISP_TX            |
+----------|-----------------------+
|   6      |     WIFI_TX           |
+----------|-----------------------+
|   7      |     WIFI_RX           |
+----------|-----------------------+
|   8      |     WIFI_EN           |
+----------|-----------------------+
...
省略
...
+----------|-----------------------+
|   47     |     DVP_PCLK          |
+----------|-----------------------+
>>> 
>>> 
>>> board_info.pin_map(8)
|---Pin----|-----Function----------|
|   8      |     WIFI_EN           |
+----------|-----------------------+
>>> board_info.WIFI_EN
8
>>> 
```

## M5StickVのGroveコネクタのピン配置(pinout)
| Color | Pin | Function |
| :---: | :---: | :---: |
| BLACK | GND | GND |
| RED | VCC | VCC |
| WHITE| IO35 | SDA/NC/DIO/DCKI |
| YELLOW | IO34 | SCL/SIG/CLK/DI |


## 参照URL

[Maixduino(schematic)](http://dl.sipeed.com/MAIX/HDK/Maixduino/Maixduino-4.30/Maixduino-4.30%28schematic%29.pdf)   
[K210 Deep Learning MCU #51](https://github.com/NirViaje/nirviaje.github.io/issues/51)   
[MaixPy Documentation](https://maixpy.sipeed.com/en/)   
[The MicroPython project](https://github.com/micropython/micropython)     
[Quick reference for the pyboard](http://docs.micropython.org/en/latest/pyboard/quickref.html)   
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)    
[MicroPython pyboard v1.1](https://www.switch-science.com/catalog/3488/)   
[M5StickV](https://www.switch-science.com/catalog/5700/)   
[Sipeed Maixduino](https://www.switch-science.com/catalog/5707/)   
[Grove - LED Socket Kit](http://wiki.seeedstudio.com/Grove-LED_Socket_Kit/)  
[GROVE - LED 赤](https://www.switch-science.com/catalog/1254/)   
[GROVE - LED 青](https://www.switch-science.com/catalog/1251/)   


以上



