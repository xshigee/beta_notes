
2020/2/2

NUCLEO F446RE MicroPython I2C接続確認
# NUCLEO F446RE MicroPython I2C接続確認

## 概要
「NUCLEO F446RE」のMicropythonをI2Cの動作確認する方法について説明する。
(ホストPCとしてはubuntuを想定している)

## 事前準備
(1)ampyのインストール   
AMPY_PORTは、自分の環境に合わせる。
```bash

pip install adafruit-ampy
export AMPY_PORT=/dev/ttyACM0
export AMPY_BAUD=115200
```
(2)picocomのインストール
```bash

sudo apt-get install picocom
```
以上のうち、exportしているものは、.bashrcに登録することを勧める。


## 参照URL
[The MicroPython project](https://github.com/micropython/micropython)     
[ＳＴＭ３２　Ｎｕｃｌｅｏ　Ｂｏａｒｄ　ＳＴＭ３２Ｆ４４６ＲＥ](http://akizukidenshi.com/catalog/g/gM-10176/)   
[Quick reference for the pyboard](http://docs.micropython.org/en/latest/pyboard/quickref.html)
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)    
[MicroPython pyboard v1.1](https://www.switch-science.com/catalog/3488/)   
[新しいボードへMicroPythonを対応させる方法をざっくり解説](https://blog.boochow.com/article/mp-stm-nucleo-l432kc.html)   


## FirmwareにバグがいるのでPatchをかけて再ビルド
I2C関係のピンアサインにバグがいるので  
Micropython/ports/stm32/boards/NUCLEO_F446RE/mpconfigboard.h   
を以下のように修正する：

```c++
// I2C busses
#define MICROPY_HW_I2C1_SCL (pin_B6)        // Arduino D10, pin 17 on CN10
#define MICROPY_HW_I2C1_SDA (pin_B7)        //              pin 21 on CN7
↓以下のように修正する
// I2C busses (same as NUCLEO_F401RE)
#define MICROPY_HW_I2C1_SCL (pin_B8)        // Arduino D15, pin 3 on CN10
#define MICROPY_HW_I2C1_SDA (pin_B9)        //         D14, pin 5 on CN10
```
修正後、再度ビルドしてファームウェアを書き込む。
この方法については以下を参照のこと。  
[NUCLEO F446RE MicroPython インストール方法](https://beta-notes.way-nifty.com/blog/2020/01/post-459022.html)   

## 動作確認
Nucleo-F446RE + Grove-Base-Shield-V2で、I2CのコネクタにI2Cのデバイスを接続する。
(ここでは、Grove-LCD-RGB-backlightを接続した)  
picocomを使いボードとシリアルで通信する。

以下、通信例：
```bash

$ picocom /dev/ttyACM0 -b115200

# 以下のようにi2c.scan()で接続しているデバイスのアドレスが出力されればＯＫ
# F446RE
>>> from machine import I2C
>>> i2c=I2C(1)
>>> i2c.scan()
[62, 98, 112] # Grove-LCD-RGB

```

以上
