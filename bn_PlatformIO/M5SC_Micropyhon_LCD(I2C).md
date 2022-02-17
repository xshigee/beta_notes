
2020/2/4

M5StickC MicroPython LCD(I2C)　demo
# M5StickC MicroPython LCD(I2C) demo

## 概要
M5StickCのGroveコネクタにLCD(I2C)を接続し、文字を表示する。

## 参照URL

[ESP32/ESP8266 Analog Readings with MicroPython](https://randomnerdtutorials.com/esp32-esp8266-analog-readings-micropython/)    
[ESP32/ESP8266 PWM with MicroPython – Dim LED](https://randomnerdtutorials.com/esp32-esp8266-pwm-micropython/)   
[ESP32/ESP8266 Digital Inputs and Digital Outputs with MicroPython](https://randomnerdtutorials.com/esp32-esp8266-digital-inputs-digital-outputs-micropython/)   
[ESP32/WROOM32](https://i0.wp.com/randomnerdtutorials.com/wp-content/uploads/2018/08/esp32-pinout-chip-ESP-WROOM-32.png)   
[M5StickC非公式日本語リファレンス(pinoutあり)](https://lang-ship.com/reference/unofficial/M5StickC/)　　
   
[Grove - LCD RGB Backlight](https://www.seeedstudio.com/Grove-LCD-RGB-Backlight.html)   

[Grove  Sensor  Grove - Rotary Angle Sensor](http://wiki.seeedstudio.com/Grove-Rotary_Angle_Sensor/)   
[Grove 温度センサ - Grove Temperature Sensor](https://jp.seeedstudio.com/Grove-Temperature-Sensor.html)   

## 関連モジュールのインストール
以下のようにソースを入手しパッチを当ててからモジュールとしてインストールする。

```bash
git clone https://github.com/Bucknalla/micropython-i2c-lcd.git
cd micropython-i2c-lcd/lib

ls
i2c_lcd.py  i2c_lcd_screen.py ...
# 上の２つのファイルにパッチを当てる
i2c_lcd.py:
14行目付近の「i2c=...」をコメントアウトする。「#  i2c = I2C(0, I2C.MASTER)」
i2c_lcd_screen.py:
53行目付近の「i2c.init(...」をコメントアウトする。「#  i2c.init(I2C.MASTER, baudrate=20000)」

#以下の３つのファイルをインストールする。
ampy put i2c_lcd.py
ampy put i2c_lcd_backlight.py
ampy put i2c_lcd_screen.py

```

## デモ・スクリプト
M5StickCのGroveコネクタに「Grove - LCD RGB Backlight」を接続し、LCDに文字を表示する。


M5SC_demo_i2c_LCD.py
```python

import i2c_lcd
from machine import Pin,I2C
from time import sleep

i2c = I2C(scl=Pin(33), sda=Pin(32), freq=100000)

d = i2c_lcd.Display(i2c)

d.home()
d.write('LCD I2C test .py')
d.move(0,1)
d.write('Hello World        ')
d.color(0xFF,0,0)
sleep(0.5)
d.color(0,0xFF,0)
sleep(0.5)
d.color(0,0,0xFF)
sleep(0.5)
d.color(0x44,0xFF,0x44)

i2c.deinit()
```


## 実行

```bash

ampy run M5SC_demo_i2c_LCD.py 

```
原因は、まだ不明だが、ときどきampyで正常動作しない場合がある。そのときは、Cloudにプログラムをコピー&ペーストして
実行するほうが確実のようだ。Cloudに接続後に/dev/ttyUSB0が復活したりするので
なにかしら初期化の問題かもしれない。

以上
 
