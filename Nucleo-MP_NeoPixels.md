
2020/3/9

Nucleo MicroPython NeoPixel demo
# Nucleo MicroPython NeoPixel demo

## 概要
Nucleo-MicroPythonボードにNeoPixelを接続する。今回、使用するモジュールは
ハードウェアSPIを利用したもので、pyboard/Nucleo用であるが、
ハードウェアSPIを持ったボードであれば、該当モジュールの多少の変更で適用可能と思われる。


## 関連モジュールのインストール
以下の手順でインストールする：
```python

git clone https://github.com/JanBednarik/micropython-ws2812.git
cd micropython-ws2812
ampy put ws2812.py


```

## デモ・スクリプト
Nucleo-MicroPythonボードに以下のようにneopixelを接続する。

接続表：
| neopixel | Nucleo/Arduino-Pin |
| :--: | :--: |
| DIN | D4 |
| VCC | 3.3V or 5V |
| GND | GND |


sample1:
```python

# connect the following pin to DIN of nexpixels
# D11 (if spi_bus=1) or D4 (if spi_bus=3)
from ws2812 import WS2812
chain = WS2812(spi_bus=3, led_count=4)
data = [
    (255, 0, 0),    # red
    (0, 255, 0),    # greenp
    (0, 0, 255),    # blue
    (85, 85, 85),   # white
]
chain.show(data)

```

sample2:
```python

from time import sleep
import math
from ws2812 import WS2812

ring = WS2812(spi_bus=3, led_count=16, intensity=0.1)

def data_generator(led_count):
    data = [(0, 0, 0) for i in range(led_count)]
    step = 0
    while True:
        red = int((1 + math.sin(step * 0.1324)) * 127)
        green = int((1 + math.sin(step * 0.1654)) * 127)
        blue = int((1 + math.sin(step * 0.1)) * 127)
        data[step % led_count] = (red, green, blue)
        yield data
        step += 1

for data in data_generator(ring.led_count):
    ring.show(data)
    sleep(0.01)

```

## 参照URL

[MicroPython WS2812 driver](https://github.com/JanBednarik/micropython-ws2812)  
[circuitpython-neopixels-using-spi](https://learn.adafruit.com/circuitpython-neopixels-using-spi)  

[M5Stack用NeoPixel互換 LEDテープ 10 cm](https://www.switch-science.com/catalog/5208/)    

[NUCLEO F446RE MicroPython インストール方法](https://beta-notes.way-nifty.com/blog/2020/01/post-459022.html)   


以上
 
