
2020/7/2

CPY XIAO MCP9808
# CPY XIAO MCP9808

## 概要
XIAOで以下のMCP9808(温度センサ)を使用する(CircuitPython版)。
CircuitPythonのインストールについては「[XIAOにCircuitPythonをインストールする](https://beta-notes.way-nifty.com/blog/2020/06/post-a69c8b.html)」を参照のこと。
(ホストPCとしてはubuntuを想定している)

[Adafruit MCP9808 Precision I2C Temperature Sensor Guide](https://learn.adafruit.com/adafruit-mcp9808-precision-i2c-temperature-sensor-guide)   


## 接続
以下の接続でボードを接続する：
| XIAO | MCP9808 |
| ----: | :--- |
| 3V3 | Vdd |
| GND | GND |
| SDA(D4) | SDA |
| SCL(D5) | SCL |
The default I2C address is 0x18

## 外部ライブラリのインストール方法
```
adafruitのアーカイブから以下のものを
CIRCUITPY/libにコピーする：

adafruit_mcp9808.mpy
adafruit_bus_device(ディレクトリ)

```

## デモ・プログラム

testmcp9808.py
```python

import time
import board
import busio
import adafruit_mcp9808

i2c_bus = busio.I2C(board.SCL, board.SDA)

# To initialise using the default address:
mcp = adafruit_mcp9808.MCP9808(i2c_bus)

# To initialise using a specified address:
# Necessary when, for example, connecting A0 to VDD to make address=0x19
# mcp = adafruit_mcp9808.MCP9808(i2c_bus, address=0x19)


while True:
    tempC = mcp.temperature
    tempF = tempC * 9 / 5 + 32
    print("Temperature: {} C {} F ".format(tempC, tempF))
    time.sleep(2)

```

## 出力例
```
picocom /dev/ttyACM0 -b115200

...
<省略>
...
Press any key to enter the REPL. Use CTRL-D to reload.
Adafruit CircuitPython 5.3.0 on 2020-04-29; Seeeduino XIAO with samd21g18
>>> import testmcp9808
Temperature: 31.5 C 88.7 F 
Temperature: 31.4375 C 88.5875 F 
Temperature: 31.5 C 88.7 F 
Temperature: 31.5 C 88.7 F 
Temperature: 31.5 C 88.7 F 
Temperature: 31.5625 C 88.8125 F 
Temperature: 31.5 C 88.7 F 
...
...

```

## 参考情報

[Seeeduino XIAO](http://akizukidenshi.com/catalog/g/gM-15178/)   

[XIAO Schematic](https://files.seeedstudio.com/wiki/Seeeduino-XIAO/res/Seeeduino-XIAO-v1.0-SCH-191112.pdf)  
[XIAO Pinout](https://www.electronics-lab.com/wp-content/uploads/2020/01/Seeeduino-XIAO-pinout.jpg)  
  
[Seeeduino XIAO Get Started By Nanase](https://wiki.seeedstudio.com/Seeeduino-XIAO-by-Nanase/)  
[コインサイズ Arduino互換機 Seeeduino XIAO を使ってみた](https://qiita.com/nanase/items/0fed598975c49b1d707e#spi-microsd%E3%82%AB%E3%83%BC%E3%83%89)  

以上
