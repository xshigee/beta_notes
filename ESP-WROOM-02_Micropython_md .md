
2020/1/13

ESP-WROOM-02 MicroPython Install
# ESP-WROOM-02 MicroPython Install

## 概要
「ESP-WROOM-02 Arduino互換ボード」にMicroPythonのfirewareを書き込む方法について述べる。
ここでは、linux環境でのインストール方法について説明する。   

## 準備
ツールをインストール前に環境整備として以下を設定する：  
(1)esp-open-sdkのインストール   
firewareの書き込みだけでは、sdkは不要だが、今後のことを考えてsdkをインストールする。
```bash

sudo apt-get install make unrar-free autoconf automake libtool gcc g++ gperf flex bison texinfo gawk ncurses-dev libexpat-dev python-dev python python-serial sed git unzip bash help2man wget bzip2 libtool-bin

mkdir mp
cd mp
git clone --recursive https://github.com/pfalcon/esp-open-sdk.git

cd esp-open-sdk
make
# 30分以上、時間がかかるので、ひたすら、待つ。

# sdkのディレクトリにパスを通す
export PATH=~/mp/esp-open-sdk/xtensa-lx106-elf/bin:$PATH

```

(2)ampyのインストール   
AMPY_PORTは、自分の環境に合わせる。
```bash

pip install adafruit-ampy
export AMPY_PORT=/dev/ttyUSB0
export AMPY_BAUD=115200
```

(3)piccomのインストール
```bash

sudo apt-get install piccom
```
以上のうち、exportしているものは、.bashrcに登録することを勧める。

## firewareのダウンロード＆書き込み
```bash

wget http://micropython.org/resources/firmware/esp8266-20191220-v1.12.bin

esptool.py --port /dev/ttyUSB0 erase_flash

esptool.py --port=/dev/ttyUSB0 --baud  460800 write_flash --flash_size=32m 0x0000 esp8266-20191220-v1.12.bin
```

## 動作確認
(1)簡単な確認
```bash

picocom /dev/ttyUSB0 -b115200

ログ出力：
>>> list(5 * x + y for x in range(10) for y in [4, 2, 1])
[4, 2, 1, 9, 7, 6, 14, 12, 11, 19, 17, 16, 24, 22, 21, 29, 27, 26, 34, 32, 31, 39, 37, 36, 44, 42, 41, 49, 47, 46]
>>>　Ctrl-D 
MPY: soft reboot
MicroPython v1.12 on 2019-12-20; ESP module with ESP8266
Type "help()" for more information.
>>> 

```
MicroPythonから抜けるには、強引なやり方になるが、USBケーブルをいったん抜いて電源を切る。

(2)プログラムの実行

```bash

ampy run blink.py
```

実際に実行するプログラムは以下になる：  
blink.py
```python

from machine import Pin
import time

# Blue LED on board
io12 = Pin(14, Pin.OUT)

while True:
    io12.value(1)
    time.sleep(0.3)
    io12.value(0)
    time.sleep(0.3)
```
ボード上の青いLEDが点滅すれば正常動作となる。

## 既に書き込んであるboot.pyの内容の確認
```python

ampy get boot.py
# This file is executed on every boot (including wake-boot from deepsleep)
#import esp
#esp.osdebug(None)
import uos, machine
#uos.dupterm(None, 1) # disable REPL on UART(0)
import gc
#import webrepl
#webrepl.start()
gc.collect()

```

## ボードにプログラムを書き込む(自動起動になる)
```bash

ampy put blink.py /main.py 
# 実行したいプログラムをmain.pyとして書き込む
# ボードの[reset]を押して起動する

#以下で書き込んであるプログラムが確認できる
ampy ls
/boot.py
/main.py
```


## 参考情報  

[ESP-WROOM-02 Arduino互換ボード](https://www.switch-science.com/catalog/2620/)   
[lvidarte/esp8266](https://github.com/lvidarte/esp8266/wiki/MicroPython:-Examples)   
[MicroPython tutorial for ESP826](https://docs.micropython.org/en/v1.9.2/esp8266/esp8266/tutorial/index.html)   
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)　　　

以上
