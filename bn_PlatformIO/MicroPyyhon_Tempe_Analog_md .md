
2020/1/25+

MicroPython Temperature Sensor(Analog)
# MicroPython Temperature Sensor(Analog)

## 概要
MicroPythonに温度センサー(アナログ)を接続する方法について説明する。
microbitのMicroPythonと他のMicroPythonで方法が異なるので、２つに分けて説明する。
ここでは、linux環境でのインストール方法について説明する。   

## 準備
(1)microbit-MicroPython用ツール(ufs)のインストール
```bash

pip install microfs
pip install --no-cache --upgrade micro
```

(2)他のMicroPython用ツール(ampy)のインストール   
AMPY_PORTは、自分の環境に合わせる。
```bash

pip install adafruit-ampy
export AMPY_PORT=/dev/ttyUSB0
export AMPY_BAUD=115200
```

(3)piccomのインストール
MicroPyhonのコンソールとして使用する。
```bash

sudo apt-get install piccom
```
以上のうち、exportしているものは、.bashrcに登録することを勧める。

## microbit MicroPythonの場合
microbit+Grove_ShieldのSlot[P0/P14]に温度センサー(アナログ)を接続する。

```bash

# サンプルログラムの書き込み
ufs tempe_ana.py main.py

# 書き込みが終わったら[reset]を押してプログラムを起動する。

```

サンプルプログラム   
tempe_ana.py
```python

from microbit import *

import math

def a2tempe(a):
   B = 4275 # v1.2 (B value of the thermistor)
   # B = 3975 # v1.0 (B value of the thermistor)
   R0 = 100000 #  R0 = 100k
   #
   R = 1023.0/a-1.0
   R = R0*R
   # convert to temperature via datasheet
   tempe  = 1.0/(math.log(R/R0)/B+1/298.15)-273.15
   return tempe

while True:
   print(a2tempe(pin0.read_analog()))
   display.scroll(a2tempe(pin0.read_analog()))
   sleep(100)

```

## Nucleo_F446REのMicroPythonの場合
Nucleo_F446RE + Grove_SheildのAnalog_Slot(A2/A1)に温度センサー(アナログ)を接続する。

```bash

# サンプルプログラムをフラッシュに書き込む
ampy put mp_tempe_ana.py /flash/main.py

# 書き込みが終わったら[reset]を押してプログラムを起動する

picocom /dev/ttyACM0 -b115200
#[reset]を押して再起動する

# 以下、出力ログ
17.24823
17.12119
17.30469
17.41757
17.31879
17.48816
17.36115
17.30469
17.30469
Ctrl-C

```

サンプルプログラム   
mp_tempe_ana.py
```python

from pyb import ADC
import machine

import time
import math

# use A1 for analog input (Nucleo_F446RE)
adc = ADC(machine.Pin('A1'))

def a2tempe(a):
   B = 4275 # v1.2 (B value of the thermistor)
   # B = 3975 # v1.0 (B value of the thermistor)
   R0 = 100000 #  R0 = 100k
   #
   # R = 4095.0/a-1.0
   vcnv = 4095*5/3.3
   R = vcnv/a-1.0
   R = R0*R
   # convert to temperature via datasheet
   tempe  = 1.0/(math.log(R/R0)/B+1/298.15)-273.15
   return tempe

while True:
   a1 = adc.read()
   print(a2tempe(a1))
   time.sleep(0.5)

```
センサーには、v1.2とv1.0の２種類が存在するので、バージョンによって
Ｂの値を変更すること。   
オリジナルのarduinoのサンプルプログラムから、
ADCのレンジが0-1023から0-4095に変更になり
電圧が5Vから3.3Vに変更になったので、
それに対応した修正を入れた。
計算式が理解できていないので、間違っている可能性があるが
とりあえず、それらしい数字(温度)が出ている。

## 使用するボードのADC(Analog Digital Converter)の仕様確認方法
同じMicroPythonといえども、ＡＤＣのボード仕様に依存しているので、実際のところ仕様が分かりにくい。
その場合、Grove-Rotary_Angle_Sensorなどを単なるボリュームとして使用して、ゼロとフルボリュームのときの
ADCの出力結果を確認する。たとえば、MicroBitではフルボリュームでは1023(@3.3V)になり、F446REでは、3000(@3.3V)くらいの値になる。このように仕様を確認して、必要があればプログラムを、それに合わせて仕様変更する。

## 参考情報  

[Nucleo-F446RE pin layout](https://os.mbed.com/platforms/ST-Nucleo-F446RE/)   
[Grove - Temperature Sensor V1.2](http://wiki.seeedstudio.com/Grove-Temperature_Sensor_V1.2/)  
[Grove - Temperature Sensor V1.0](https://seeeddoc.github.io/Grove-Temperature_Sensor/)   
[Grove - Rotary Angle Sensor](http://wiki.seeedstudio.com/Grove-Rotary_Angle_Sensor/)   
[Grove - 温度や湿度センサー](http://wiki.seeedstudio.com/jp/Grove-TemperatureAndHumidity_Sensor/)   
[Grove - DHT11](https://jp.seeedstudio.com/Grove-Temperature-Humidity-Sensor-DHT11.html)   
[micro:bit用GROVEシールド v2.0](https://www.switch-science.com/catalog/5434/)   
[Grove - Base Shield v1.2](https://seeeddoc.github.io/Grove-Base_Shield_v1.2/)  
[MicroFS](https://github.com/ntoll/microfs)  [ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)   
　　

以上
