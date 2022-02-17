
2020/2/16

CC3200 MicroPython examples
# CC3200 MicroPython examples

## 概要
CC3200 MicroPythonのサンプルについてまとめた。   
なお、基板上のピン番号(Pxx)とGPIOなどの対応は以下のソースコードの以下にある：   
micropython/ports/cc3200/boards/cc3200_af.csv  


## Peformace Test for CC3200
```python

from machine import RTC
rtc = RTC(datetime=(2020, 2, 9, 10, 11, 0, 0, None)) 

def performanceTest():
     secs = rtc.now()[5] 
     endTime = secs + 10
     count = 0
     while rtc.now()[5] < endTime:
         count += 1
     print("Count: ", count)
performanceTest()
```
以下のような出力が出る   
Count:  5529  
他のMicroPythonボードに比べて遅い結果だが、省電優先設計のせいだと思われる。


# Digital Out
```python

from machine import Pin
# Orange LED - D6(GP10) on board
gp10_out = Pin('GP10', mode=Pin.OUT)
gp10_out.value(1)
gp10_out.value(0)
# Green LED  - D5(GP11) on board
gp11_out = Pin('GP11', mode=Pin.OUT)
gp11_out.value(1)
gp11_out.value(0)
```


# Digital In

```python

from machine import Pin
# SW2 GPIO22
# SW3 GPIO13
# make GP22 an input with the pull-up enabled
gp22_in = Pin('GP22', mode=Pin.IN, pull=Pin.PULL_UP)
gp22_in() # get value, 0 or 1 on SW2
# make GP13 an input with the pull-up enabled
gp13_in = Pin('GP13', mode=Pin.IN, pull=Pin.PULL_UP)
gp13_in() # get value, 0 or 1 on SW3
```

# SW control LED
```python

from machine import Pin
ledOrange = Pin('GP10', mode=Pin.OUT)
ledGreen = Pin('GP11', mode=Pin.OUT)
sw2 = Pin('GP22', mode=Pin.IN, pull=Pin.PULL_UP)
sw3 = Pin('GP13', mode=Pin.IN, pull=Pin.PULL_UP)
while True:
   ledOrange.value(sw2())
   ledGreen.value(sw3())

```

# Analog In
```python

from time import sleep_ms
# p57 GP2 a#0
# p58 GP3 a#1
# p59 GP4 a#2
# p60 GP5 a#3 
# read value, 0-4095
from machine import ADC
adc = ADC()
a0in = adc.channel(pin='GP2')
a1in = adc.channel(pin='GP3')
a2in = adc.channel(pin='GP4')
a3in = adc.channel(pin='GP5')
while True:
   print('p57:'+str(a0in()))
   print('p58:'+str(a1in()))
   print('p59:'+str(a2in()))
   print('p60:'+str(a3in()))
   print('============')
   sleep_ms(1000)
```


# PWM
```python

# make LEDs off
from machine import Pin
# Orange LED - D6(GP10) on board
gp10_out = Pin('GP10', mode=Pin.OUT)
gp10_out.value(0)
# Green LED  - D5(GP11) on board
gp11_out = Pin('GP11', mode=Pin.OUT)
gp11_out.value(0)
#
from machine import Timer
# timer 3 in PWM mode and width must be 16 buts
tim = Timer(3, mode=Timer.PWM, width=16)
# enable channel A @1KHz with a 1.55% duty cycle
tim_a = tim.channel(Timer.A, freq=1000, duty_cycle=0155) # p1(GP10): Orange LED on board
# enable channel B @1KHz with a 30.55% duty cycle
tim_b = tim.channel(Timer.B, freq=1000, duty_cycle=3055) # p2(GP11): Green LED on board
```

# I2C
```python
from machine import Pin, I2C
scl0 = Pin('GP10', mode=Pin.ALT, alt=1) 
sda0 = Pin('GP11', mode=Pin.ALT, alt=1)
i2c = I2C(0, scl=scl0, sda=sda0)
i2c.scan()
# [24, 65] (ボード上の以下のデバイスのアドレスが返る)
# BMA222　(加速度センサ)  24(0x18)
# TMP006　(温度センサ) 65(0x41)
```

# SDCard(動作確認無し、ピン配置からの推測)
```python

# SDCard
from machine import SD, Pin
import os

# clock pin, cmd pin, data0 pin
# 以下の２つの設定のうち１つを使用する

# 設定＃１
sd0_clk = Pin('GP10', mode=Pin.ALT, alt=6) 
sd0_cmd = Pin('GP11', mode=Pin.ALT, alt=6)
sd0_dat0 = Pin('GP15', mode=Pin.ALT, alt=8)

# 設定#2
sd0_clk = Pin('GP16', mode=Pin.ALT, alt=8) 
sd0_cmd = Pin('GP17', mode=Pin.ALT, alt=8)
sd0_dat0 = Pin('GP15', mode=Pin.ALT, alt=8)

sd = SD(pins=(sd0_clk, sd0_cmd, sd0_dat0) # SD0_CLK(SCL) SD0_CMD(MOSI) SD0_DAT0(MISO)

os.mount(sd, '/sd')

```

以上
