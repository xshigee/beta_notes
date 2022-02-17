
2020/3/7

ESP-WROOM-02 MicroPython TMP102(i2c)
# ESP-WROOM-02 MicroPython TMP102(i2c)

## 概要
「ESP-WROOM-02 Arduino互換ボード」(ESP8266)のMicroPythonに温度センサーTMP102(i2c)を接続する。
ここでは、linux環境でのインストール方法について説明する。   

## 配線表
以下のように配線する。

| Arduino_Pin | ESP8266_Pin | TEMP102 |
| :--: | :--: | :--: |
| SCL | IO05 | SCL |
| SDA | IO04 | SDA |
| 3V3 | 3V3 | VCC |
| GND | GND | GND |

## 関連モジュールのインストール
以下の手順でインストールする：
```bash

git clone https://github.com/khoulihan/micropython-tmp102.git
cd micropython-tmp102
cp _tmp102.py tmp102.py
ampy put tmp102.py

```
関連モジュールもあるが、必要なコアとなるモジュールのみをインストールする。

## 動作確認用スクリプト

ESP8266_TMP102_test.py
```python

# ESP8266
import utime
from machine import Pin, I2C
from tmp102 import Tmp102
i2c = I2C(scl=Pin(5), sda=Pin(4), freq=100000)
sensor = Tmp102(i2c, 0x48)

while True:
   print('Temperature: {0:.1f}'.format(sensor.temperature))
   utime.sleep(2)

```

## 出力結果(REPL画面)

```python

Temperature: 19.6
Temperature: 19.6
Temperature: 19.6
Temperature: 19.6
Temperature: 20.8
Temperature: 22.5
# 動作確認のためにセンサーを手で触れたので
# 急に温度が上がっている

```

## Nucleoの場合のスクリプト

Nucleo_TMP102_test.py  
```python

# Nucleo
import utime
from machine import Pin, I2C
from tmp102 import Tmp102
i2c = I2C(1)
sensor = Tmp102(i2c, 0x48)

while True:
   print('Temperature: {0:.1f}'.format(sensor.temperature))
   utime.sleep(2)

```
「i2c = ...」のところのみが変更になる。


## 参考情報  

[TMP102搭載 デジタル温度センサモジュール](https://www.switch-science.com/catalog/3161/)  

[ESP-WROOM-02 Arduino互換ボード](https://www.switch-science.com/catalog/2620/)   
[lvidarte/esp8266](https://github.com/lvidarte/esp8266/wiki/MicroPython:-Examples)   
[MicroPython tutorial for ESP826](https://docs.micropython.org/en/v1.9.2/esp8266/esp8266/tutorial/index.html)   
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)　　　

以上
