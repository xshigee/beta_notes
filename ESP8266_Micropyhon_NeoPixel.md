
2020/2/5

ESP8266 MicroPython NeoPixel demo
# ESP8266 MicroPython NeoPixel demo

## 概要
MicroPythonボードのGroveコネクタにNeoPxelを接続する。

## 参照URL

使用機材：  
[M5Stack用NeoPixel互換 LEDテープ 10 cm](https://www.switch-science.com/catalog/5208/)    
[ESP-WROOM-02 Arduino互換ボード](https://www.switch-science.com/catalog/2620/)   
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
内蔵モジュールneopixelを使用するので、インストールするものはない。

## デモ・スクリプト
「ESP-WROOM-02 Arduino互換ボード(ESP8266)」 +  Grove-Base-Sheild のSlot_D3に[M5Stack用NeoPixel互換 LEDテープ 10 cm]を接続する。(実際には、D4に接続することになる）

動作確認用のスクリプト  
コンソールに入り手打ちで動作を確認する：
```python

# neopixel simple test
from machine import Pin
from neopixel import NeoPixel
pin = Pin(2, Pin.OUT)  # D4 (Arduino Pin) @Grove Solet_D3
np = NeoPixel(pin, 8) 
np[0] = (255, 255, 255)
np[1] = (255, 255, 0)
np[2] = (255, 0, 0)
np[3] = (0, 255, 255)
np[4] = (0, 255, 0)
np[5] = (0, 0, 255)
np[6] = (255, 255,0)
np[7] = (255, 0, 0)
np.write()

```

ESP8266_demo_NeoPixel.py   
(MicroPythonのdocにあったもの)  
```python

import time

def demo(np):
    n = np.n

    # cycle
    for i in range(4 * n):
        for j in range(n):
            np[j] = (0, 0, 0)
        np[i % n] = (255, 255, 255)
        np.write()
        time.sleep_ms(25)

    # bounce
    for i in range(4 * n):
        for j in range(n):
            np[j] = (0, 0, 128)
        if (i // n) % 2 == 0:
            np[i % n] = (0, 0, 0)
        else:
            np[n - 1 - (i % n)] = (0, 0, 0)
        np.write()
        time.sleep_ms(60)

    # fade in/out
    for i in range(0, 4 * 256, 8):
        for j in range(n):
            if (i // 256) % 2 == 0:
                val = i & 0xff
            else:
                val = 255 - (i & 0xff)
            np[j] = (val, 0, 0)
        np.write()

    # clear
    for i in range(n):
        np[i] = (0, 0, 0)
    np.write()
#----------------------
from machine import Pin
from neopixel import NeoPixel
pin = Pin(2, Pin.OUT)  # D4 (Arduino Pin) @Grove Solet_D3
np = NeoPixel(pin, 15)
demo(np)

```


## 実行

```bash

# RAM実行
ampy run ESP8266_demo_NeoPixel.py

```

以上
 
