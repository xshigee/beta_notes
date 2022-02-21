
2020/5/26  

TinyGO Install Circuit-Playground-Express
# TinyGO Install Circuit-Playground-Express

## 概要
以下のCircuit-Playground-ExpressボードでTinyGOを動かす。
以降、Circuit-Playground-ExpressをCPXとする。
(ホストPCとしてはubuntuを想定している)

[Circuit Playground Express](https://www.switch-science.com/catalog/3666/)   


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
CPXでは使用しない。

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

# for CPX(Circuit-Playground-Express)
alias tgcpx="docker run -it --rm -v $PWD:/go/src/github.com/myusr/myapp -w /go/src/github.com/myusr/myapp -e GOPATH=/go tinygo/tinygo:latest tinygo build -size full -o cpx.uf2 -target circuitplay-express ."
alias fcpx="cp cpx.uf2 /media/USER_NAME/CPLAYBOOT/"
# for nucleo-f103rb
alias tgf103="docker run -it --rm -v $PWD:/go/src/github.com/myusr/myapp -w /go/src/github.com/myusr/myapp -e GOPATH=/go tinygo/tinygo:latest tinygo build -size full -o firmware.bin -target nucleo-f103rb ."
alias stf="st-flash write firmware.bin 0x8000000"
```
「USER_NAME」の部分は自分の環境に合わせること。
なお、この設定には別ボード用も含む。  

以上のexportは、.bashrcに登録する。

## bootloader mode
CPXにfirmwareを書き込めるモードを「bootloader mode」といい、このモードでは、USBストレージとしてCPLAYBOOTが現れる。
CPXに事前に、どんなプログラムが書かれていたかで、やり方が異なる。
arduinoのプログラムかCircuitPythonのプログラムが書き込まれている場合、resetをdouble-clickのように２度押すと
このモードに入りUSBストレージとしてCPLAYBOOTが現れる。その他の場合、resetを1度押すと、このモードに入る。

ただし、本家サイトの説明ではdouble-clickのようにと説明があるが
実際には、２度目のリセットは少し長めに押してから離さないとbootloader-modeには入らない。
このモードに入るとボード上のnexpixelが緑になる。  
失敗すると赤色になる。その場合、再度トライすること.

工場出荷の状態では、arduinoのデモプログラムが入っているようなので
実際にはresetの２度押しで、このモードに入る。

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

tgcpx
```
これで、cpx.uf2が作成される。

出力例：
```bash

$ tgcpx
   code  rodata    data     bss |   flash     ram | package
   3442      13       0       0 |    3455       0 | (bootstrap)
     60       0       0       0 |      60       0 | github
      0      36       0       0 |      36       0 | handleHardFault$string
    108      24       0       4 |     132       4 | internal/task
   2992      43       5    2054 |    3040    2059 | machine
      0      16       0     130 |      16     130 | machine$alloc
   2302      79       0      45 |    2381      45 | runtime
     96       0       0       0 |      96       0 | time
   9000     211       5    2233 |    9216    2238 | (sum)
   9540       -       8    4312 |    9548    4320 | (all)

$ ls -l cpx.*
-rwxr-xr-x 1 root root 19456  5月 25 10:35 cpx.uf2

```

(4)書き込み   
以下を実行して書き込む：
```bash

# ボードとホストPCをUSBで接続する。
# resetを２度押して、bootloader-modeに入る。
# CPLAYBOOTのディレクトリが現れる。
fcpx

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

tgcpx

# ボードとホストPCをUSBで接続する。
# resetを２度押して、bootloader-modeに入る。
# CPLAYBOOTのディレクトリが現れる。
fcpx

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

var leds [10]color.RGBA

func main() {
	led := machine.LED
	led.Configure(machine.PinConfig{Mode: machine.PinOutput})

	neo := machine.NEOPIXELS
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
tgcpx
# ボードをbootloader-modeに入れる
fcpx
```
書き込みが終わると、ボードのnexpixelが色を変えて光る。
　
## 温度/光センサを動かす

```bash

mkdir thermistor
cd thermistor

cp -r $TIGOLIBS/thermistor .

cp $TIGOLIBS/examples/thermistor/main.go .

# main.goを編集する
leafpad main.go
# 以下のように修正する：
(光センサの部分も追加している)
```
```go

// This example uses the settings for the thermistor that is built in to the
// Adafruit Circuit Playground Express.
package main

import (
	"machine"
	"time"

	"./thermistor"
	//"tinygo.org/x/drivers/thermistor"
)

func main() {
	machine.InitADC()

	lsense := machine.ADC{machine.LIGHTSENSOR} // Light Sensor (set ADC pin)
	lsense.Configure() // start the pin's ADC function

	sensor := thermistor.New(machine.TEMPSENSOR) // Tempe Sensor (set ADC pin)
	sensor.Configure()

	for {
		temp, _ := sensor.ReadTemperature()
		println("Temperature:", temp/1000, "°C")		
		println("Light Sensor:", lsense.Get())

		time.Sleep(1 * time.Second)
	}
}
```
続き：
```bash
# 以下、ビルド&書き込み
. ~/setenv4tg
tgcpx
# bootloader-modeに入る
fcpx

```
実行後、「picocom /dev/ttyACM0 -b115200」を実行して、シリアル出力を表示すると、温度と光センサの値が出力される。

## 加速度センサ(LIS3DH(i2c))を動かす

```bash

mkdir lis3dh
cd lis3dh

cp -r $TIGOLIBS/lis3dh .
cp $TIGOLIBS/examples/lis3dh/main.go .

leafpad main.go
# 以下の内容にする：
(修正点はimportの部分のみ)
```
```go

// Connects to a LIS3DH I2C accelerometer on the Adafruit Circuit Playground Express.
package main

import (
	"machine"
	"time"

	"./lis3dh"
	//"tinygo.org/x/drivers/lis3dh"
)

var i2c = machine.I2C1

func main() {
	i2c.Configure(machine.I2CConfig{SCL: machine.SCL1_PIN, SDA: machine.SDA1_PIN})

	accel := lis3dh.New(i2c)
	accel.Address = lis3dh.Address1 // address on the Circuit Playground Express
	accel.Configure()
	accel.SetRange(lis3dh.RANGE_2_G)

	println(accel.Connected())

	for {
		x, y, z, _ := accel.ReadAcceleration()
		println("X:", x, "Y:", y, "Z:", z)

		rx, ry, rz := accel.ReadRawAcceleration()
		println("X (raw):", rx, "Y (raw):", ry, "Z (raw):", rz)

		time.Sleep(time.Millisecond * 100)
	}
}

```
続き：
```bash
# 以下、ビルド&書き込み
. ~/setenv4tg
tgcpx
# bootloader-modeに入る
fcpx

```
実行後、「picocom /dev/ttyACM0 -b115200」を実行して、シリアル出力を表示すると、加速度センサの値が出力される。

出力例：
```bash

$ picocom /dev/ttyACM0 -b115200
...
Terminal ready
X: -110378 Y: -16605 Z: 958241
X (raw): -1984 Y (raw): -144 Z (raw): 16064
X: -136752 Y: -22466 Z: 999267
X (raw): -2240 Y (raw): -368 Z (raw): 15920
X: -136752 Y: -13675 Z: 993406
X (raw): -2224 Y (raw): -320 Z (raw): 16272

```

## 音センサ(マイク)を動かす

```bash

mkdir lis3dh
cd lis3dh

cp -r $TIGOLIBS/microphone .
cp $TIGOLIBS/examples/microphone/main.go .

leafpad main.go
# 以下の内容にする：
(修正点はimportの部分のみ)
```
```go

// Example using the i2s hardware interface on the Adafruit Circuit Playground Express
// to read data from the onboard MEMS microphone.
//
// Uses ideas from the https://github.com/adafruit/Adafruit_CircuitPlayground repo.
//
package main

import (
	"machine"

	"./microphone"
	//"tinygo.org/x/drivers/microphone"
)

const (
	defaultSampleRate        = 22000
	quantizeSteps            = 64
	msForSPLSample           = 50
	defaultSampleCountForSPL = (defaultSampleRate / 1000) * msForSPLSample
)

func main() {
	machine.I2S0.Configure(machine.I2SConfig{
		Mode:           machine.I2SModePDM,
		AudioFrequency: defaultSampleRate * quantizeSteps / 16,
		ClockSource:    machine.I2SClockSourceExternal,
		Stereo:         true,
	})

	mic := microphone.New(machine.I2S0)
	mic.SampleCountForSPL = defaultSampleCountForSPL
	mic.Configure()

	for {
		spl, maxval := mic.GetSoundPressure()
		println("C", spl, "max", maxval)
	}
}
```
続き：
```bash
# 以下、ビルド&書き込み
. ~/setenv4tg
tgcpx
# bootloader-modeに入る
fcpx

```
実行後、「picocom /dev/ttyACM0 -b115200」を実行して、シリアル出力を表示すると、音センサ(マイク)の値が出力される。

出力例：
```bash

$ picocom /dev/ttyACM0 -b115200
...
C 85536 max 43
C 100324 max 236
C 85735 max 44
C 85735 max 44
C 85735 max 44
C 85536 max 43
C 104530 max 383
C 88990 max 64
C 85536 max 43
C 89125 max 65
C 85536 max 43
C 106178 max 463
C 85536 max 43
C 85735 max 44
C 85735 max 44
C 98973 max 202
C 85735 max 44
...

```

## 外部のtmp102(i2c)を動かす

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
tgcpx
# bootloader-modeに入る
fcpx

```
CPXにtmp102(i2c)を接続して、プログラム実行後、「picocom /dev/ttyACM0 -b115200」を実行して、シリアル出力を表示すると、tmp102(温度)の値が出力される。


## CPXのピン定義
```

const (
	LED		= D13
	NEOPIXELS	= D8

	BUTTONA	= D4
	BUTTONB	= D5
	SLIDER	= D7	// built-in slide switch

	BUTTON	= BUTTONA
	BUTTON1	= BUTTONB

	LIGHTSENSOR	= A8
	TEMPSENSOR	= A9
	PROXIMITY	= A10
)

Capacitive Touch: A1,A2,A3,A4,A5,A6,A7

Infrared Receive and Transmit:
IR TX: D29
IR RX: D39
(The direct analog value is available from pin A10)

const (
	SDA_PIN	= PB02	// I2C0 external
	SCL_PIN	= PB03	// I2C0 external

	SDA1_PIN	= PA00	// I2C1 internal
	SCL1_PIN	= PA01	// I2C1 internal
)

```

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


以上

