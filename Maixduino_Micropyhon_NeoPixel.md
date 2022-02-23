
2020/2/5

Maixduino MicroPython NeoPixel demo
# Maixduino MicroPython NeoPixel demo

## 概要
MaixduinoボードのGroveコネクタにNeoPxelを接続する。

## 参照URL

使用機材：  
[M5Stack用NeoPixel互換 LEDテープ 10 cm](https://www.switch-science.com/catalog/5208/)    
[Sipeed Maixduino](https://www.switch-science.com/catalog/5707/)  
[Arduino用 GroveベースシールドV2 - Grove Base Shield V2](https://jp.seeedstudio.com/Base-Shield-V2.html)  


ESP32/ESP8266関係:  
[ESP32/ESP8266 Analog Readings with MicroPython](https://randomnerdtutorials.com/esp32-esp8266-analog-readings-micropython/)    
[ESP32/ESP8266 PWM with MicroPython – Dim LED](https://randomnerdtutorials.com/esp32-esp8266-pwm-micropython/)   
[ESP32/ESP8266 Digital Inputs and Digital Outputs with MicroPython](https://randomnerdtutorials.com/esp32-esp8266-digital-inputs-digital-outputs-micropython/)   
[ESP32/WROOM32](https://i0.wp.com/randomnerdtutorials.com/wp-content/uploads/2018/08/esp32-pinout-chip-ESP-WROOM-32.png)   

その他：  
[NUCLEO F446RE MicroPython インストール方法](https://beta-notes.way-nifty.com/blog/2020/01/post-459022.html)   
[STM32F4-Discovery」にMicroPythonをインストールする](https://beta-notes.way-nifty.com/blog/2020/02/post-6822aa.html)  
[Grove - 温度および湿度センサー (DHT11)](https://jp.seeedstudio.com/Grove-Temperature-Humidity-Sensor-DHT11.html)  
[Grove 温度および湿度センサPro – DHT22 / AM2302 - Grove Temperature & Humidity Sensor Pro](https://jp.seeedstudio.com/Grove-Temperature-Humidity-Sensor-Pro-AM2302.html)   
  [Grove - LCD RGB Backlight](https://www.seeedstudio.com/Grove-LCD-RGB-Backlight.html)   
[Grove  Sensor  Grove - Rotary Angle Sensor](http://wiki.seeedstudio.com/Grove-Rotary_Angle_Sensor/)   
[Grove 温度センサ - Grove Temperature Sensor](https://jp.seeedstudio.com/Grove-Temperature-Sensor.html)   

## 関連モジュールのインストール
内蔵モジュールws2812を使用するので、インストールするものはない。

## デモ・スクリプト
Maixuuino +  Grove-Base-Sheild のSlot_D3に[M5Stack用NeoPixel互換 LEDテープ 10 cm]を接続する。(実際には、D4に接続することになる）

動作確認用のスクリプト  
コンソールに入り手打ちで動作を確認する：  
(np.display()が実行されるまでＬＥＤは点灯しないので注意のこと)
```python

# neopixel simple test
from modules import ws2812
from board import board_info
np = ws2812(board_info.D[4],8) # D4 (Arduino Pin) @Grove Slot_D3
np.set_led(0,(255, 255, 255))
np.set_led(1,(255, 255, 0))
np.set_led(2,(255, 0, 0))
np.set_led(3,(0, 255, 255))
np.set_led(4,(0, 255, 0))
np.set_led(5,(0, 0, 255))
np.set_led(6,(255, 255,0))
np.set_led(7,(255, 0, 0))
np.display()

```

MXD_demo_NeoPixel.py   
(ESP8266のMicroPythonのdocにあったものを移植した)  
```python

# Maixduino
import time

def demo(np,n):
    # cycle
    for i in range(4 * n):
        for j in range(n):
             sink = np.set_led(j,(0,0,0))
        sink = np.set_led(i % n,(255, 255, 255))
        sink = np.display()
        time.sleep_ms(25)

    # bounce
    for i in range(4 * n):
        for j in range(n):
            sink = np.set_led(j,(0,0,128))
        if (i // n) % 2 == 0:
            sink = np.set_led(i % n,(0, 0, 0))
        else:
            sink = np.set_led(n - 1 - (i % n),(0, 0, 0))
        sink = np.display()
        time.sleep_ms(60)

    # fade in/out
    for i in range(0, 4 * 256, 8):
        for j in range(n):
            if (i // 256) % 2 == 0:
                val = i & 0xff
            else:
                val = 255 - (i & 0xff)
            sink = np.set_led(j,(val, 0, 0))
        sink = np.display()

    # clear
    for i in range(n):
        sink = np.set_led(i,(0, 0, 0))
    sink = np.display()
#----------------------
from modules import ws2812
from board import board_info
np = ws2812(board_info.D[4],15) # D4 (Arduino Pin) @Grove Slot_D3
demo(np,15)

```


## 実行

```bash

# RAM実行
ampy run MXD_demo_NeoPixel.py

```

## 注意
ボードを抜き差しすると、/dev/ttyUSB0が/dev/ttyUSB1などに変更になるので注意のこと。  
「export AMPY_PORT=...」で変更する必要がある。


以上
 
