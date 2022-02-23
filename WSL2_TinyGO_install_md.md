
2021/3/25  

WSL2 TinyGO Install
# WSL2 TinyGO Install

## 概要
WSL2でTinyGOを動かす。   
WSL2のUbuntu20.04をホストにする前提で    
XIAOボードを中心としてインストールからビルド書き込みまで説明している。  
target名を変更することで他のボードのビルドなども行うことができる。

## Go/TinyGoのインストール
Ubuntu20.04/WSL2で以下を実行する：

```bash

mkdir ~/tigo_ws

cd ~/Downloads

# goのインストール
wget https://golang.org/dl/go1.16.2.linux-amd64.tar.gz

sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.16.2.linux-amd64.tar.gz

export PATH=$PATH:/usr/local/go/bin

# tinygoのインストール
wget https://github.com/tinygo-org/tinygo/releases/download/v0.17.0/tinygo_0.17.0_amd64.deb
sudo dpkg -i tinygo_0.17.0_amd64.deb

export PATH=$PATH:/usr/local/tinygo/bin

# その他のexport
export GOPATH=$HOME/tigo_ws
export TIGOLIBS=$GOPATH/pkg/mod/tinygo.org/x/drivers@v0.15.1

# 書き込み用のディレクトリを作る
mkdir /mnt/c/temp
```

以下を.bashrcに追加する：
```

# tinygo/go
export PATH=$PATH:/usr/local/go/bin
export PATH=$PATH:/usr/local/tinygo/bin
export GOPATH=$HOME/tigo_ws
export TIGOLIBS=$GOPATH/pkg/mod/tinygo.org/x/drivers@v0.15.1
```
## 書き込みスクリプト
WSL2はUSBデバイス(USBストレージも含む)をサポートしていないので、  
WSL2でhexまたはuf2のファイルまで作成した後、  
PowerShell.exe経由で
USBストレージに書き込むことで、ボード書き込みを行う。

このスクリプトは、「~/tigo_ws」に置いて、実際のプロジェクト・ディレクトリにコピーして使用することとする。  
以下、対応ボードごとのbashスクリプトになる。  

fUF2.sh
```bash

#!/bin/bash
rm /mnt/c/temp/*
cp *.uf2 /mnt/c/temp/
PowerShell.exe copy c:/temp/*.uf2 e:
exit 0
```
・実行属性を与えること。  
・「e:」は環境依存なので、自分の環境で、uf2ファイルを書き込むドライブ・レターを設定すること。  

fHEX.sh
```bash

#!/bin/bash
rm /mnt/c/temp/*
cp *.hex /mnt/c/temp/
PowerShell.exe copy c:/temp/*.hex e:
exit 0
```
・実行属性を与えること。  
・「e:」は環境依存なので、自分の環境で、uf2ファイルを書き込むドライブ・レターを設定すること。  


## インストール時付属サンプルのビルド例

```

cd ~/tigo_ws

# microbit系
tinygo build -size full -o=mb.hex -target=microbit examples/microbit-blink
tinygo build -size full -o=mb2.hex -target=microbit-v2 examples/microbit-blink

# SAMD系
tinygo build -size full -o=cpx.uf2 -target=circuitplay-express examples/blinky1 
tinygo build -size full -o=fm4.uf2 -target=feather-m4 examples/blinky1 
tinygo build -size full -o=xiao.uf2 -target=xiao examples/blinky1 
tinygo build -size full -o=wiot.uf2 -target=wioterminal examples/blinky1 

# mbed系
tinygo build -size full -o f103.hex -target nucleo-f103rb examples/blinky1
```

## 独自プロジェクト(blinky)の作成
独自プロジェクト(blinky)を作成するには  
ディレクトリblinkyを作成して、  

(1)その中に以下のファイルを作成する：     
blinky.go
```go

package main

// This is the most minimal blinky example and should run almost everywhere.

import (
	"machine"
	"time"
)

func main() {
	led := machine.LED
	led.Configure(machine.PinConfig{Mode: machine.PinOutput})
	for {
		led.Low()
		time.Sleep(time.Millisecond * 1000)

		led.High()
		time.Sleep(time.Millisecond * 1000)
	}
}
```

(2)go.modの作成  
以下を実行してgo.modを作成する：
```

go mod init blinky
```
「blinky」は任意のようだが
プロジェクト名(ディレクトリ名)に合わせる。

(3)ビルド  
以下を実行してビルドする：
```bash

tinygo build -size full -o=xiao.uf2 -target=xiao  .
```
これで、xiao.uf2が作成される。

出力例：
```bash

$ tinygo build -size full -o=xiao.uf2 -target=xiao  .
   code  rodata    data     bss |   flash     ram | package
   2574      13       0     130 |    2587     130 | (bootstrap)
      0      36       0       0 |      36       0 | handleHardFault$string
    164      24       4       4 |     192       8 | internal/task
   2844      18      13    2054 |    2875    2067 | machine
      0      16       0       0 |      16       0 | machine$alloc
     40       0       0       0 |      40       0 | main
   2352      79       0      48 |    2431      48 | runtime
     28       0       0       0 |      28       0 | runtime/volatile
     92       0       0       0 |      92       0 | time
   8094     186      17    2236 |    8297    2253 | (sum)
   8508       -      20    4312 |    8528    4332 | (all)


$ ls -l *.uf2
-rw-r--r-- 1 xxxx xxxx 17408  3月 25 08:10 xiao.uf2
```

(4)書き込み   
以下を実行して書き込む：
```bash

# 書き込みスクリプトをカレント・ディレクトリにコピーする
cp ~/tigo_ws/fUF2.sh .

# ボードとホストPCをUSBで接続する。
# RSTパッドを２度GNDショートして、bootloader-modeに入る。
# Arduinoのディレクトリが現れる。

#書き込み
./fUF2.sh
```
ボードの赤いLED(D13)が１秒周期で点滅すれば動作としてはＯＫとなる。

## serialを動かす
(1)以下のようにプロジェクト用ディレクトリを作成する：
```bash

cd tigo_ws
mkdir serial
cd serial

```
(2)プロジェクト(serial)ディレクトリに  
以下のファイルを作成する：   
serial.go
```go

package main

import "time"

func main() {
	for {
		println("hello world!")
		time.Sleep(time.Second)
	}
}
```

(3)go.modの作成  
以下を実行してgo.modを作成する：
```

go mod init serial
```
「serial」は任意のようだが
プロジェクト名(ディレクトリ名)に合わせる。

(4)ビルド/書き込み
以下を実行する：
```bash

# ビルド
tinygo build -size full -o=xiao.uf2 -target=xiao  .

# 書き込みスクリプトをカレント・ディレクトリにコピーする
cp ~/tigo_ws/fUF2.sh .

# ボードとホストPCをUSBで接続する。
# RSTパッドを２度GNDショートして、bootloader-modeに入る。
# Arduinoのディレクトリが現れる。

#書き込み
./fUF2.sh
```
以上で、プログラムが起動する。

(4)シリアル通信の起動  
windows側の通信ソフト(teratermなど)でシリアル接続すると  
以下のような出力が得られる。(通信速度：115200 bps)
```bash

hello world!
hello world!
hello world!
hello world!
hello world!
hello world!
hello world!
hello world!
hello world!
...

```

## TinyGOのモジュールを利用する

```bash

実行例：

# 以下で、モジュールをインストールする
go get tinygo.org/x/drivers

# インストールしたモジュールのパスを設定する
#(この場合は設定済みなので特にやることはない)

#　インポートしたいモジュールを確認する
ls $TIGOLIBS
#出力
CHANGELOG.md     bmp280        hcsr04      microbitmatrix  st7789
CONTRIBUTING.md  bmp388        hd44780     microphone      tester
LICENSE          buzzer        hd44780i2c  mma8653         thermistor
Makefile         dht           hub75       mpu6050         tmp102
README.md        displayer.go  i2c.go      net             touch
adt7410          drivers.go    ili9341     pcd8544         veml6070
adxl345          ds1307        l293x       semihosting     version.go
amg88xx          ds3231        l9110x      shifter         vl53l1x
apa102           easystepper   lis2mdl     shiftregister   waveshare-epd
at24cx           espat         lis3dh      sht3x           wifinina
bh1750           examples      lsm303agr   spi.go          ws2812
blinkm           flash         lsm6ds3     ssd1306
bme280           go.mod        mag3110     ssd1331
bmi160           go.sum        mcp23017    ssd1351
bmp180           gps           mcp3008     st7735

#　ここでは例として「ws2812」を使用してみる
mkdir ws2812
cd ws2812

# exampleのmain.goをコピーする
cp $TIGOLIBS/examples/ws2812/main.go .

chmod +rw main.go

# main.goを編集する
nano main.go
# 以下のように修正する：
```
```go

// Connects to an WS2812 RGB LED strip with 10 LEDS.
//
// See either the others.go or digispark.go files in this directory
// for the neopixels pin assignments.
package main

import (
	"image/color"
	"machine"
	"time"

	"tinygo.org/x/drivers/ws2812"
)

// 以下は接続するneopixelの数に合わせる
var leds [64]color.RGBA
//var leds [10]color.RGBA

func main() {
	led := machine.LED
	led.Configure(machine.PinConfig{Mode: machine.PinOutput})

        neo := machine.D4 // for Feater-M4/XIAO
//        neo := machine.NEOPIXELS

	neo.Configure(machine.PinConfig{Mode: machine.PinOutput})

	ws := ws2812.New(neo)
	rg := false

	for {
		rg = !rg
		for i := range leds {
			rg = !rg
			if rg {
				// Alpha channel is not supported by WS2812 so we leave it out
				//leds[i] = color.RGBA{R: 0xff, G: 0x00, B: 0x00}
				leds[i] = color.RGBA{R: 0x0f, G: 0x00, B: 0x00}
			} else {
				//leds[i] = color.RGBA{R: 0x00, G: 0xff, B: 0x00}
				leds[i] = color.RGBA{R: 0x00, G: 0x0f, B: 0x00}
			}
		}

		ws.WriteColors(leds[:])
		led.Set(rg)
		time.Sleep(100 * time.Millisecond)
	}
}
```
続き：
```bash

# go.modを初期化する
go mod init ws2812
# 以下のコマンドで必要なモジュールを自動設定する
go mod tidy

# ビルド
tinygo build -size full -o=xiao.uf2 -target=xiao  .

# 書き込みスクリプトをカレント・ディレクトリにコピーする
cp ~/tigo_ws/fUF2.sh .

# ボードとホストPCをUSBで接続する。
# RSTパッドを２度GNDショートして、bootloader-modeに入る。
# Arduinoのディレクトリが現れる。

#書き込み
./fUF2.sh
```
D4をneopixelsのDIN,3V3をVCC,GNDをGNDに接続する。   
書き込んで実行すると、接続したneopixelsが色を変えて光る。

　
## tmp102(i2c)を動かす

```bash

mkdir tmp102
cd tmp102

cp $TIGOLIBS/examples/tmp102/main.go .
chmod a+rw main.go

nano main.go
# 以下の内容にする：
(修正点は無いはずだが以下の内容であることを確認する)
```
```go

package main

import (
	"fmt"
	"machine"
	"time"

	"tinygo.org/x/drivers/tmp102"
)

func main() {
	machine.I2C0.Configure(machine.I2CConfig{
		Frequency: machine.TWI_FREQ_400KHZ,
	})

	thermo := tmp102.New(machine.I2C0)
	thermo.Configure(tmp102.Config{})

	for {

		temp, _ := thermo.ReadTemperature()

		print(fmt.Sprintf("%.2f°C\r\n", float32(temp)/1000.0))

		time.Sleep(time.Millisecond * 1000)
	}

}
```
続き：
```bash

# go.modを設定する
go mod init tmp102
go mod tidy

# 以下、ビルド&書き込み
tinygo build -size full -o=xiao.uf2 -target=xiao  .

# 書き込みスクリプトをカレント・ディレクトリにコピーする
cp ~/tigo_ws/fUF2.sh .

# ボードとホストPCをUSBで接続する。
# RSTパッドを２度GNDショートして、bootloader-modeに入る。
# Arduinoのディレクトリが現れる。

#書き込み
./fUF2.sh
```

## XIAOのピン定義

| port | pin |
| ---: | :--- |
| PA02 | A0/D0|
| PA04 | A1/D1|
| PA10 | A2/D2|
| PA11 | A3/D3 |
| PA08 | A4/D4|
| PA09 | A5/D5 |
| PB08 | A6/D6 |
| PB09 | A7/D7 |
| PA07 | A8/D8 |
| PA05 | A9/D9 |
| PA06 | A10/D10 |


## ボードのターゲット名
「-target=nxiao」を以下にあるターゲット名に変更すると  
 他のボード用にビルドできる。

主なもの(ビルド時にhexまたはuf2を生成するもの)
```

(1)Adafruit Circuit Playground Bluefruit  
circuitplay-bluefruit  

(2)Adafruit Circuit Playground Express  
circuitplay-express  

(3)Adafruit Feather M0  
feather-m0  

(4)Adafruit Feather M4  
feather-m4  

(5)Adafruit ItsyBitsy M0  
itsybitsy-m0  

(6)Adafruit ItsyBitsy M4  
itsybitsy-m4  

(7)BBC micro:bit  
microbit  

(8)BBC micro:bit v2 
microbit-v2  

(9)seeed XIAO
xiao

(10)seeed wio-terminal
wioterminal

(11)ST Micro "Nucleo F103RB"  
nucleo-f103rb  
```
・mbed系(nucleo,microbitなど)のボードは、hexファイルで書き込むので、   
書き込みの際は、fHEX.shを使用すること。  
・上で上げているadafruitならびにseeedのボードは、uf2ファイルで書き込むので、   
書き込みの際は、fUF2.shを使用すること。

## 参考情報
(古い情報も含まれているので注意のこと)  

https://github.com/tinygo-org/tinygo  
https://tinygo.org/  
https://tinygo.org/getting-started/linux/  

[コンピュータボードでTinyGOを動かす](https://beta-notes.way-nifty.com/blog/2020/04/post-1ecc01.html)   
[docker/TinyGO Helper Script](https://beta-notes.way-nifty.com/blog/2020/04/post-4cd82c.html)   
[TinyGOでLightSensorを動かす](https://beta-notes.way-nifty.com/blog/2020/04/post-78050e.html)  

[TinyGoで始める組み込みプログラミング](https://tech.144lab.com/entry/tinygo)  
[TinyGo on Arduino Uno: An Introduction](https://create.arduino.cc/projecthub/alankrantas/tinygo-on-arduino-uno-an-introduction-6130f6)  

[Circuit Playground Express](https://www.switch-science.com/catalog/3666/)   
[Adafruit Circuit Playground Express - Overview](https://learn.adafruit.com/adafruit-circuit-playground-express)  
[Infrared Receive and Transmit with Circuit Playground Express](https://learn.adafruit.com/infrared-ir-receive-transmit-circuit-playground-express-circuit-python/overview)  

[Adafruit Circuit Playground Express - PINOUT](https://learn.adafruit.com/adafruit-circuit-playground-express/pinouts)  
[Adafruit Circuit Playground Express　Pin Assign](https://tinygo.org/microcontrollers/machine/circuitplay-express/)  

[NUCLEO-F103RB mbed pinout](https://os.mbed.com/platforms/ST-Nucleo-F103RB/)   
[NUCLEO-F103RB　Pin Assign](https://tinygo.org/microcontrollers/machine/nucleo-f103rb/)  
[STM32F4DISCO　Pin Assign](https://tinygo.org/microcontrollers/machine/stm32f4disco/)  
[MICROBIT Pin Assign](https://tinygo.org/microcontrollers/machine/microbit/)  
[ARDUINO-NANO Pin Assign](https://tinygo.org/microcontrollers/machine/arduino-nano/)   
[ARDUINO Pin Assign](https://tinygo.org/microcontrollers/machine/arduino/)  

[TinyGo Drivers](https://github.com/tinygo-org/drivers/)  

[USB Flashing Format (UF2)](https://github.com/Microsoft/uf2)   

[XIAO Schematic(zip)](https://files.seeedstudio.com/wiki/Seeeduino-XIAO/res/Seeeduino-XIAO-v1.0.zip)   
[How to use Seeeduino XIAO to log in to your Raspberry PI](https://wiki.seeedstudio.com/How-to-use-Seeeduino-XIAO-to-log-in-to-your-Raspberry-PI/)  

WSL2関連：  
[WSL2でRDPサーバーを動かす](https://beta-notes.way-nifty.com/blog/2021/03/post-bd6ce0.html)  
[WSL2でSSHサーバーを動かす](https://beta-notes.way-nifty.com/blog/2021/03/post-7e6528.html)  
[WSL2でのplatformioやPico-SDKの利用方法](https://beta-notes.way-nifty.com/blog/2021/02/post-72506a.html)  


以上

