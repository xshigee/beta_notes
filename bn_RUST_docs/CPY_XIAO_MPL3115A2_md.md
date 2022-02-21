
2020/7/2

CPY XIAO MPL115A2
# CPY XIAO MPL115A2

## 概要
XIAOで以下のMPL3115A2(気圧・高度・気温センサ)を使用する(CircuitPython版)。
CircuitPythonのインストールについては「[XIAOにCircuitPythonをインストールする](https://beta-notes.way-nifty.com/blog/2020/06/post-a69c8b.html)」を参照のこと。
(ホストPCとしてはubuntuを想定している)

[MPL115A2 - I2C Barometric Pressure/Temperature Sensor](https://www.adafruit.com/product/992)   


## 接続
以下の接続でボードを接続する：
| XIAO | MPL3115A2 |
| ----: | :--- |
| 3V3 | Vin |
| GND | GND |
| SDA(D4) | SDA |
| SCL(D5) | SCL |
The default I2C address is 0x60

## 外部ライブラリのインストール方法
```

adafruitのアーカイブから以下のものを
CIRCUITPY/libにコピーする：

adafruit_mpl3115a2.mpy
adafruit_bus_device(ディレクトリ)
```

## デモ・プログラム

testmpl3115a2.py
```python

# Simple demo of the MPL3115A2 sensor.
# Will read the pressure and temperature and print them out every second.
# Author: Tony DiCola
import time
import board
import busio
import adafruit_mpl3115a2

# Initialize the I2C bus.
i2c = busio.I2C(board.SCL, board.SDA)

# Initialize the MPL3115A2.
sensor = adafruit_mpl3115a2.MPL3115A2(i2c)
# Alternatively you can specify a different I2C address for the device:
# sensor = adafruit_mpl3115a2.MPL3115A2(i2c, address=0x10)

# You can configure the pressure at sealevel to get better altitude estimates.
# This value has to be looked up from your local weather forecast or meteorlogical
# reports.  It will change day by day and even hour by hour with weather
# changes.  Remember altitude estimation from barometric pressure is not exact!
# Set this to a value in pascals:
sensor.sealevel_pressure = 102250

# Main loop to read the sensor values and print them every second.
while True:
    pressure = sensor.pressure
    print("Pressure: {0:0.3f} hPa".format(pressure/100))
    altitude = sensor.altitude
    print("Altitude: {0:0.3f} meters".format(altitude))
    temperature = sensor.temperature
    print("Temperature: {0:0.3f} degrees Celsius".format(temperature))
    print("");
    time.sleep(1.0)

```

## 出力例
```
picocom /dev/ttyACM0 -b115200

...
<省略>
...


Press any key to enter the REPL. Use CTRL-D to reload.
Adafruit CircuitPython 5.3.0 on 2020-04-29; Seeeduino XIAO with samd21g18
>>> import testmpl3115a2
Pressure: 1001.597 hPa
Altitude: 173.003 meters
Temperature: 30.187 degrees Celsius

Pressure: 1001.575 hPa
Altitude: 172.753 meters
Temperature: 30.187 degrees Celsius

Pressure: 1001.632 hPa
Altitude: 172.753 meters
Temperature: 30.187 degrees Celsius

Pressure: 1001.580 hPa
Altitude: 173.128 meters
Temperature: 30.187 degrees Celsius

...

```

## 参考情報

[Seeeduino XIAO](http://akizukidenshi.com/catalog/g/gM-15178/)   

[XIAO Schematic](https://files.seeedstudio.com/wiki/Seeeduino-XIAO/res/Seeeduino-XIAO-v1.0-SCH-191112.pdf)  
[XIAO Pinout](https://www.electronics-lab.com/wp-content/uploads/2020/01/Seeeduino-XIAO-pinout.jpg)  

[Seeeduino XIAO Get Started By Nanase](https://wiki.seeedstudio.com/Seeeduino-XIAO-by-Nanase/)  
[コインサイズ Arduino互換機 Seeeduino XIAO を使ってみた](https://qiita.com/nanase/items/0fed598975c49b1d707e#spi-microsd%E3%82%AB%E3%83%BC%E3%83%89)  

以上
