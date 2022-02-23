
2020/4/19

TinyGO LightSensor
# TinyGO LightSensor

## 概要
TinyGOで以下のLightSensorを動かす。ここでは、
ターゲットボードとして、Arduino-UNOを使用する。
(ホストPCとしてはubuntuを想定している)

[Grove - Light Sensor](https://seeeddoc.github.io/Grove-Light_Sensor/)   


## 準備
以下のツールを予めインストールする：  
(1)arduino関係
```bash

sudo apt-get install gcc-avr
sudo apt-get install avr-libc
sudo apt-get install avrdude

```
(2)Docker/TinyGOを最新版にする
```bash

docker pull tinygo/tinygo:latest

```

(3)build&flashのスクリプトを用意する(UNO用)    
以下のファイルをカレント・ディレクトリに実行属性を与えて用意する：  
fsuno.sh 
```bash

#!/bin/bash
rm -f uno.hex
docker run --rm -v $(pwd):/src -v $GOPATH:/go -e "GOPATH=/go" tinygo/tinygo:latest tinygo build  -o /src/uno.hex -target=arduino $1
if test -f ./uno.hex; then
  avrdude -p atmega328p -C ~/.platformio/packages/tool-avrdude/avrdude.conf -c arduino -b 115200 -D -P /dev/ttyACM0 -U flash:w:uno.hex:i
  picocom /dev/ttyACM0 -b19200
fi
```
プログラム(TinyGO)のなかのprintlnに対応するために、プログラム書き込み後、自動的にpicocomを起動する。

実行属性を与えるには以下を実行する：
```bash

chmod +x fsuno.sh

```


## プログラム
作業用ディレクトリとして以下を作成する：
```bash

cd $GOPATH/src
mkdir adc0
cd adc0

```
カレント・ディレクトリに以下のプログラム(ファイル)を用意する：  
adc0.go
```go

package main

import (
    "machine"
    "time"
)

func main() {

    machine.InitADC() // initialize ADC
    lsense := machine.ADC{4} // Light Sensor (set ADC pin)
    lsense.Configure() // start the pin's ADC function

    //led := machine.Pin(13)  // LED on board
    led := machine.Pin(8) // D8 connected to LED
    led.Configure(machine.PinConfig{1})

    for {
        // output analog reading to serial port
        println("Light:",lsense.Get())
        // light up led if analog reading is over the threshold
        led.Set(lsense.Get() < 5000)
        delay(100)
    }
}

func delay(t int64) {
    time.Sleep(time.Duration(1000000 * t))
}
```
LightSensorは、A4に接続する。
LEDは、D8に接続する。
または、プログラムを変更して、
オンボードのLEDを使用する。

## ビルド/書き込み/実行
ボードを接続して以下を実行する：
```bash

./fsuno.sh adc0

```

## 出力
書き込み完了後、以下のような出力がホストのターミナルに出力される:
```bash

Light: 21056
Light: 25856
Light: 23936
Light: 22720
```
LightSensorを手などで光を遮ると
LEDが光る。

## 参考情報

https://github.com/tinygo-org/tinygo  
https://tinygo.org/  
https://tinygo.org/getting-started/linux/  

[TinyGoで始める組み込みプログラミング](https://tech.144lab.com/entry/tinygo)
[TinyGo on Arduino Uno: An Introduction](https://create.arduino.cc/projecthub/alankrantas/tinygo-on-arduino-uno-an-introduction-6130f6)  

[STM32F4DISCO　Pin Assign](https://tinygo.org/microcontrollers/machine/stm32f4disco/)  
[MICROBIT Pin Assign](https://tinygo.org/microcontrollers/machine/microbit/)  
[ARDUINO-NANO Pin Assign](https://tinygo.org/microcontrollers/machine/arduino-nano/)   
[ARDUINO Pin Assign](https://tinygo.org/microcontrollers/machine/arduino/)  

[TinyGo Drivers](https://github.com/tinygo-org/drivers/)  


以上

