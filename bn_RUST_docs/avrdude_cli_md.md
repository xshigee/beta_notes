
2020/4/15  

avrdude cli
# avrdude cli

## 概要
arduinoの書き込みツールであるavrdudeをcli(command line interface)で使用する。
cliで与えるパラメータなどが分からなかったので、そこを明確にするためのメモである。
(ホストPCとしてはubuntuを想定している)

## avrduinoを使った書き込み(arduino-nanoの場合)
以下で書き込むことができる：
```bash

avrdude -p atmega328p -C ~/.platformio/packages/tool-avrdude/avrdude.conf -c arduino -b 57600 -D -P /dev/ttyUSB0 -U flash:w:nano.hex:i

# なお、/dev/ttyUSB0,nano.hexは、自分のボードや環境に合わせて変更すること
# avrdude.confは、Arduino-IDEのものがエラーになったので
# platformioのものを使用している。

```

## (platformioの)avrdude.confのダウンロード方法
platformioでArduinoのビルドをしていないとavrdude.confがdownloadされない。
platformioを使用していない場合は、以下の手順でplatformioをインストールしてビルドすると
(結果として)avrdude.confをdownloadできる。

```bash

pip install platformio

mkdir temp
cd temp
pio init --board nanoatmega328

nano src/main.ino
#以下の内容になるように編集する：
#(ツールを動かすためのダミー)
```
src/main.ino
```c++
void setup()
{
    Serial.begin(115200);
}

void loop()
{
    Serial.println("serial test");
    delay(1000);
}
```
続き：
```bash
# ボードを接続しておく

pio run
pio run -t nobuild -t upload -v

# プログラムを動かす必要はないが
# 動作確認をしたいなら以下を実行する：
picocom /dev/ttyUSB0 -b115200
# 出力例：
serial test
serial test
serial test
serial test
serial test
serial test
serial test

```

## 他のボードの書き込み用パラメータを知る方法

「pio run -t nobuild -t upload -v」を実行すると
実際に実行されるコマンドラインを出力するので
別のボードを使うときの書き込み方法を知ることができる。

以下、実例：
```bash
$ pio boards | grep -i sparkfun
...
sparkfun_promicro8           ATMEGA32U4     8MHz         28KB      2.50KB  SparkFun Pro Micro 3.3V/8MHz
sparkfun_promicro16          ATMEGA32U4     16MHz        28KB      2.50KB  SparkFun Pro Micro 5V/16MHz
sparkfun_qduinomini          ATMEGA32U4     8MHz         28KB      2.50KB  SparkFun Qduino Mini
sparkfun_redboard            ATMEGA328P     16MHz        31.50KB   2KB     SparkFun RedBoard
sparkfun_serial7seg          ATMEGA328P     8MHz         31.50KB   2KB     SparkFun Serial 7-Segment Display
sparkfun_samd21_9dof           SAMD21G18A   48MHz        256KB    32KB   SparkFun 9DoF Razor IMU M0
sparkfun_qwiic_micro_samd21e   SAMD21E18A   48MHz        256KB    32KB   SparkFun Qwiic Micro
sparkfun_redboard_turbo        SAMD21G18A   48MHz        256KB    32KB   SparkFun RedBoard Turbo
...

pio init --board sparkfun_qduinomini

nano src/main.ino
# ビルドまで進むようにダミーのプログラムを作る。

# ボードを接続しておく

pio run
pio run -t nobuild -t upload -v

# 出力からコマンドラインとしては以下を知ることができる：

avrdude -v -p atmega32u4 -C /home/<user_name>/.platformio/packages/tool-avrdude/avrdude.conf -c avr109 -b 57600 -D -P "/dev/ttyUSB0" -U flash:w:.pio/build/sparkfun_qduinomini/firmware.hex:i


```

## 参考情報

[Programming with AVRdude](https://learn.adafruit.com/introducing-trinket/programming-with-avrdude)  
[ArduinoISPを汎用AVRライタとして使う(2)](https://synapse.kyoto/tips/ArduinoISP_AVRWriter/page002.html)  
[ubuntuでavrdudeを使う](https://kivantium.hateblo.jp/entry/20120105/1325726060)  

以上

