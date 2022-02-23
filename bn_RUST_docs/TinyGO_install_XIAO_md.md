
2020/6/11  

TinyGO Install XIAO
# TinyGO Install XIAO

## 概要
以下のXIAOボードでTinyGOを動かす。XIAOはサポート対象のボードになっていないので、同じチップを採用しているFeather-M0としてビルドする。
(ホストPCとしてはubuntuを想定している)

[Seeeduino XIAO](http://akizukidenshi.com/catalog/g/gM-15178/)   


## 準備
以下のツールを予めインストールする：  
(1)st-flash
```bash

sudo apt install cmake
sudo apt install libusb-1.0

git clone https://github.com/texane/stlink.git
cd stlink
make
cd build/Release
sudo make install
sudo ldconfig

```
XIAOでは使用しない。

(2)TinyGO(Docker版)のインストール   
以下の手順でインストールする：
```bash

(1)dockerのインストール

sudo apt-get docker
sudo apt-get docker.io

sudo groupadd docker 
sudo usermod -aG docker $USER
sudo systemctl enable docker
sudo systemctl start docker
#ここで再起動する

(2)TinyGOのインストール

docker pull tinygo/tinygo:latest

```

(3)GOのインストール  
TinyGOのモジュールを使用するのにGOが必要なので 予めインストールする。  
(ただし、TinyGOとの整合性により最新版ではない)  
```bash

wget https://dl.google.com/go/go1.13.7.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.13.7.linux-amd64.tar.gz

export PATH=$PATH:/usr/local/go/bin
export GOPATH=$HOME/tinygo_ws
```
なお、GOPATHのパス設定値は、任意だが、それをベースに
その配下にTinyGO/GOの関係ファイル/ディレクトリが置かれる。

alias用スクリプトの設定：  
nano ~/setenv4tg   
以下の内容に編集する：
```
# for XIAO
alias tgxiao="docker run -it --rm -v $PWD:/go/src/github.com/myusr/myapp -w /go/src/github.com/myusr/myapp -e GOPATH=/go tinygo/tinygo:latest tinygo build -size full -o xiao.uf2 -target feather-m0 ."
alias fxiao="cp xiao.uf2 /media/USER_NAME/Arduino/"
```
「USER_NAME」の部分は自分の環境に合わせること。

以上のexportは、.bashrcに登録する。

## bootloader mode
XIAOにfirmwareを書き込めるモードを「bootloader mode」といい、このモードでは、USBストレージとしてArduinoが現れる。
arduinoのプログラムかCircuitPythonのプログラムが書き込まれている場合、RSTパッドを２度GNDとショートすると
このモードに入りUSBストレージとしてarduinoが現れる。

## blinkyを動かす
(1)以下のようにプロジェクト用ディレクトリを作成する：
```bash

cd tigo_ws
mkdir blinky
cd blinky

#aliasを設定する
source ~/setenv4tg
#上のコマンドはプロジェクトのディレクトリで実行する必要がある。
#「source」は「.」に置き換えることができる。

```
(2)プロジェクト(blinky)ディレクトリに
以下のファイルを作成する：   
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
(3)ビルド  
以下を実行してビルドする：
```bash

tgxiao
```
これで、xiao.uf2が作成される。

出力例：
```bash

$ tgxiao
   code  rodata    data     bss |   flash     ram | package
   3400      13       0       0 |    3413       0 | (bootstrap)
     40       0       0       0 |      40       0 | github
      0      36       0       0 |      36       0 | handleHardFault$string
    108      24       0       4 |     132       4 | internal/task
   2992      43       5    2054 |    3040    2059 | machine
      0      16       0     130 |      16     130 | machine$alloc
   2302      79       0      45 |    2381      45 | runtime
     92       0       0       0 |      92       0 | time
   8934     211       5    2233 |    9150    2238 | (sum)
   9476       -       8    4312 |    9484    4320 | (all)

$ ls -l xiao.*
-rwxr-xr-x 1 root root 19456  6月 11 09:54 xiao.uf2
```

(4)書き込み   
以下を実行して書き込む：
```bash

# ボードとホストPCをUSBで接続する。
# RSTパッドを２度GNDショートして、bootloader-modeに入る。
# Arduinoのディレクトリが現れる。
fxiao

```
ボードの赤いLED(D13)が１秒周期で点滅すれば動作としてはＯＫとなる。

## serialを動かす
(1)以下のようにプロジェクト用ディレクトリを作成する：
```bash

cd tigo_ws
mkdir serial
cd serial

#aliasを設定する
source ~/setenv4tg
#上のコマンドはプロジェクトのディレクトリで実行する必要がある

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

(3)ビルド/書き込み
以下を実行する：
```bash

tgxiao

# ボードとホストPCをUSBで接続する。
# RSTパッドを２度GNDショートして、bootloader-modeに入る。
# Arduinoのディレクトリが現れる。
fxiao

```
以上で、プログラムが起動する。

(4)シリアル通信の起動  
以下のように別の端末でシリアル通信を起動すると
シリアルに結果が出力される：
```bash
$ picocom /dev/ttyACM0 -b115200
picocom v1.7

<省略>

Terminal ready
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
export TIGOLIBS=$GOPATH/src/tinygo.org/x/drivers/

#　インポートしたいモジュールを確認する
ls $TIGOLIBS
  ...
  adt7410          ds1307        l9110x          shifter        vl53l1x
  adxl345          ds3231        lis3dh          shiftregister  waveshare-epd
  amg88xx          easystepper   lsm6ds3         sht3x          wifinina
  apa102           espat         mag3110         ssd1306        ws2812
  at24cx           examples      mcp3008         ssd1331
  bh1750           flash         microbitmatrix  st7735
  ...

#　ここでは例として「ws2812」をインポートする
mkdir ws2812
cd ws2812

# モジュールをディレクトリの形でコピーする
cp -r $TIGOLIBS/ws2812 .

# exampleのmain.goをコピーする
cp $TIGOLIBS/examples/ws2812/main.go .

# main.goを編集する
leafpad main.go
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

	"./ws2812"
	//"tinygo.org/x/drivers/ws2812"
)

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

# 以下、ビルド&書き込み
. ~/setenv4tg
tgxiao
# ボードをbootloader-modeに入れる
fxiao
```
D4をneopixelsのDIN,3V3をVCC,GNDをGNDに接続する。
書き込んで実行すると、接続したneopixelsが色を変えて光る。

　
## tmp102(i2c)を動かす

```bash

mkdir tmp102
cd tmp102

cp -r $TIGOLIBS/tmp102 .
cp $TIGOLIBS/examples/tmp102/main.go .

leafpad main.go
# 以下の内容にする：
(修正点はimportの部分のみ)
```
```go

package main

import (
	"fmt"
	"machine"
	"time"

	"./tmp102"
	//"tinygo.org/x/drivers/tmp102"
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
# 以下、ビルド&書き込み
. ~/setenv4tg
tgxiao
# bootloader-modeに入る
fxiao

```
ピン定義が実際のXIOAと異なるせいで動作しなかった。

## 総括
制限付きだがXIAOでTinyGOが動作することが確認できた。
本格的に使用するには、ピン定義を実際のXIAOのものに合わせる必要がある。

## Feather-M0のピン定義

流用しているFeather-M0のピン定義を参考に以下にあげる：
```

const (
	D0	= PA11	// UART0 RX
	D1	= PA10	// UART0 TX
	D2	= NoPin	// does not seem to exist
	D3	= PA09
	D4	= PA08
	D5	= PA15	// PWM available
	D6	= PA20	// PWM available
	D7	= NoPin	// does not seem to exist
	D8	= PA06
	D9	= PA07	// PWM available
	D10	= PA18	// can be used for PWM or UART1 TX
	D11	= PA16	// can be used for PWM or UART1 RX
	D12	= PA19	// PWM available
	D13	= PA17	// PWM available
)

const (
	A0	= PA02	// ADC/AIN[0]
	A1	= PB08	// ADC/AIN[2]
	A2	= PB09	// ADC/AIN[3]
	A3	= PA04	// ADC/AIN[4]
	A4	= PA05	// ADC/AIN[5]
	A5	= PB02	// ADC/AIN[10]
)

const (
	LED = D13
)

const (
	USBCDC_DM_PIN	= PA24
	USBCDC_DP_PIN	= PA25
)

const (
	UART_TX_PIN	= D10
	UART_RX_PIN	= D11
)

const (
	SDA_PIN	= PA22	// SDA: SERCOM3/PAD[0]
	SCL_PIN	= PA23	// SCL: SERCOM3/PAD[1]
)

const (
	SPI0_SCK_PIN	= PB11	// SCK: SERCOM4/PAD[3]
	SPI0_MOSI_PIN	= PB10	// MOSI: SERCOM4/PAD[2]
	SPI0_MISO_PIN	= PA12	// MISO: SERCOM4/PAD[0]
)
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

## 参考情報

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

以上

