
2020/2/24

RpiZero/Nucleo-MicroPython 3AxisCompass(I2C) demo  
# RpiZero/Nucleo-MicroPython 3AxisCompass(I2C) demo 

## 概要
RpiZero/Nucleo/ESP8266/ESP32にGroveコネクタ経由で「Grove - 3Axis Compass」(I2C)を接続する。
なお、今回のデバイスの場合、電源は5V/3.3Vとも動作する。

## 関連モジュールのインストール
以下を実行する：

ESP8266の場合
```bash
# ESP8266
git clone https://github.com/gvalkov/micropython-esp8266-hmc5883l.git
cd micropython-esp8266-hmc5883l/
ampy put hmc58831l.py

```

RpiZero/Nucleo/ESP32の場合
```bash
git clone https://github.com/gvalkov/micropython-esp8266-hmc5883l.git
cd micropython-esp8266-hmc5883l/
#　hmc58831l.pyを修正してhmc58831l_md.pyとする。修正点は、次の修正スクリプトを参照のこと。
# 修正したhmc58831l_md.pyをフラッシュまたはブートSDに書き込む。

```

修正スクリプト：  
修正箇所は「#」でコメントアウトした付近になる。  
hmc58831l_md.py
```python

import math
import machine

from ustruct import pack
from array import array


class HMC5883L:
    __gain__ = {
        '0.88': (0 << 5, 0.73),
        '1.3':  (1 << 5, 0.92),
        '1.9':  (2 << 5, 1.22),
        '2.5':  (3 << 5, 1.52),
        '4.0':  (4 << 5, 2.27),
        '4.7':  (5 << 5, 2.56),
        '5.6':  (6 << 5, 3.03),
        '8.1':  (7 << 5, 4.35)
    }

#    def __init__(self, scl=4, sda=5, address=30, gauss='1.3', declination=(0, 0)):
    def __init__(self, i2c, address=30, gauss='1.3', declination=(0, 0)):
#        self.i2c = i2c = machine.I2C(scl=machine.Pin(scl), sda=machine.Pin(sda), freq=100000)
        self.i2c = i2c

        # Initialize sensor.
#        i2c.start()

        # Configuration register A:
        #   0bx11xxxxx  -> 8 samples averaged per measurement
        #   0bxxx100xx  -> 15 Hz, rate at which data is written to output registers
        #   0bxxxxxx00  -> Normal measurement mode
        i2c.writeto_mem(30, 0x00, pack('B', 0b111000))

        # Configuration register B:
        reg_value, self.gain = self.__gain__[gauss]
        i2c.writeto_mem(30, 0x01, pack('B', reg_value))

        # Set mode register to continuous mode.
        i2c.writeto_mem(30, 0x02, pack('B', 0x00))
#       i2c.stop()

        # Convert declination (tuple of degrees and minutes) to radians.
        self.declination = (declination[0] + declination[1] / 60) * math.pi / 180

        # Reserve some memory for the raw xyz measurements.
        self.data = array('B', [0] * 6)

    def read(self):
        data = self.data
        gain = self.gain

        self.i2c.readfrom_mem_into(30, 0x03, data)

        x = (data[0] << 8) | data[1]
        z = (data[2] << 8) | data[3]
        y = (data[4] << 8) | data[5]

        x = x - (1 << 16) if x & (1 << 15) else x
        y = y - (1 << 16) if y & (1 << 15) else y
        z = z - (1 << 16) if z & (1 << 15) else z

        x = round(x * gain, 4)
        y = round(y * gain, 4)
        z = round(z * gain, 4)

        return x, y, z

    def heading(self, x, y):
        heading_rad = math.atan2(y, x)
        heading_rad += self.declination

        # Correct reverse heading.
        if heading_rad < 0:
            heading_rad += 2 * math.pi

        # Compensate for wrapping.
        elif heading_rad > 2 * math.pi:
            heading_rad -= 2 * math.pi

        # Convert from radians to degrees.
        heading = heading_rad * 180 / math.pi
        degrees = math.floor(heading)
        minutes = round((heading - degrees) * 60)
        return degrees, minutes

    def format_result(self, x, y, z):
        degrees, minutes = self.heading(x, y)
        return 'X: {:.4f}, Y: {:.4f}, Z: {:.4f}, Heading: {}° {}′ '.format(x, y, z, degrees, minutes)

```

## テスト・スクリプト
ボードに「Grove - 3Axis Compass」を接続し、センサーデータを出力する。

ESP8266用  
ESP8266_3AxisCompass_test.py
```python

# ESP8266
from hmc5883l import HMC5883L

sensor = HMC5883L(scl=5, sda=4)

while True:
   x, y, z = sensor.read()
   print(sensor.format_result(x, y, z))

```

Nucleo/RPiZero用  
Nucleo_3AxisCompass_test.py
```python
# Nucleo/RPiZero
from hmc5883l_md import HMC5883L
from machine import I2C

i2c=I2C(1)
sensor = HMC5883L(i2c)

while True:
   x, y, z = sensor.read()
   print(sensor.format_result(x, y, z))

```


ESP32用   
ESP32_3AxisCompass_test.py
```python
# ESP32
from hmc5883l_md import HMC5883L
from machine import Pin, I2C

i2c = I2C(scl=Pin(5), sda=Pin(4), freq=100000)
sensor = HMC5883L(i2c)

while True:
   x, y, z = sensor.read()
   print(sensor.format_result(x, y, z))

```
なお、ESP32の場合、ESP8266用モジュールを使用しても動作するはずである。


出力例：
```pyhon


X: -6.4400, Y: 402.9600, Z: 19.3200, Heading: 90° 55′ 
X: -6.4400, Y: 402.9600, Z: 19.3200, Heading: 90° 55′ 
X: 1.8400, Y: 399.2800, Z: 22.0800, Heading: 89° 44′ 
X: 1.8400, Y: 399.2800, Z: 22.0800, Heading: 89° 44′ 
X: 0.9200, Y: 398.3600, Z: 20.2400, Heading: 89° 52′ 
...

```

## 参照URL

[BareMetalのMicropythonをRaspberryPi_Zeroにインストールしてみる](https://beta-notes.way-nifty.com/blog/2020/02/post-573471.html)   
[NUCLEO F446RE MicroPython インストール方法](https://beta-notes.way-nifty.com/blog/2020/01/post-459022.html)  
[パッチ方法 - NUCLEO F446RE MicroPython I2C接続確認の方法](https://beta-notes.way-nifty.com/blog/2020/02/post-4dc66d.html)  
[ESP8266 - ESP-WROOM-02 MicroPython インストール方法](https://beta-notes.way-nifty.com/blog/2020/01/post-8e39c0.html)  


[GROVE - 4ピン-ジャンパメスケーブル (5本セット)](https://www.switch-science.com/catalog/1048/)  
[Grove 4ピンコネクタ - ジャンパーピン変換ケーブル(5本入り) (ジャンパー側オス)](https://www.sengoku.co.jp/mod/sgk_cart/detail.php?code=EEHD-4K34)  



[Grove - 3-Axis Compass](http://wiki.seeedstudio.com/Grove-3-Axis_Compass_V1.0/)  

以上
