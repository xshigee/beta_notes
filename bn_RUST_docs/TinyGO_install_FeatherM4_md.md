
2020/5/29  

TinyGO Install Feather-M4-Express
# TinyGO Install Feather-M4-Express

## 概要
以下のFeather-M4-ExpressボードでTinyGOを動かす。
以降、Feather-M4-ExpressをFeatherM4とする。
(ホストPCとしてはubuntuを想定している)

[Feather-M4-Express](https://www.switch-science.com/catalog/5887/)   


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
FeatherM4では使用しない。

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

(4)alias用スクリプトの設定  
nano ~/setenv4tg   
以下の内容に編集する：
```

# for Feather-M4-Express
alias tgfm4="docker run -it --rm -v $PWD:/go/src/github.com/myusr/myapp -w /go/src/github.com/myusr/myapp -e GOPATH=/go tinygo/tinygo:latest tinygo build -size full -o fm4.uf2 -target feather-m4 ."
alias ffm4="cp fm4.uf2 /media/USER_NAME/FEATHERBOOT/"
```
「USER_NAME」の部分は自分の環境に合わせること。  

以上のexportは、.bashrcに登録する。

## bootloader mode
FeatherM4にfirmwareを書き込めるモードを「bootloader mode」といい、このモードでは、USBストレージとしてFEATHERBOOTが現れる。
FeatherM4に事前に、どんなプログラムが書かれていたかで、やり方が異なる。
arduinoのプログラムかCircuitPythonのプログラムが書き込まれている場合、resetをdouble-clickのように２度押すと
このモードに入りUSBストレージとしてFEATHERBOOTが現れる。その他の場合、resetを1度押すと、このモードに入る。

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

tgfm4
```
これで、fm4.uf2が作成される。

出力例：
```bash

$ tgfm4
   code  rodata    data     bss |   flash     ram | package
   1304      13       0       0 |    1317       0 | (bootstrap)
     36       0       0       0 |      36       0 | github
      0      36       0       0 |      36       0 | handleHardFault$string
    102      24       0       4 |     126       4 | internal/task
   3114      27       5    2054 |    3146    2059 | machine
      0      16       0     130 |      16     130 | machine$alloc
   2276      79       0      45 |    2355      45 | runtime
    104       0       0       0 |     104       0 | time
   6936     195       5    2233 |    7136    2238 | (sum)
   7860       -       8    6360 |    7868    6368 | (all)


$ ls -l *.uf2
-rwxr-xr-x 1 root root 15872  5月 29 11:19 fm4.uf2

```

(4)書き込み   
以下を実行して書き込む：
```bash

# ボードとホストPCをUSBで接続する。
# resetを２度押して、bootloader-modeに入る。
# FEATHERBOOTのディレクトリが現れる。
ffm4

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

tgfm4

# ボードとホストPCをUSBで接続する。
# resetを２度押して、bootloader-modeに入る。
# FEATHERBOOTのディレクトリが現れる。
ffm4

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
(外部のneopixelを使用するように修正している)
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

        neo := machine.D4 // for Feater-M4
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
tgcpx
# ボードをbootloader-modeに入れる
fcpx
```
neopixelのDINをD4と接続して実行すると、neopixelが光る。  

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
tgfm4
# bootloader-modeに入る
ffm4

```
なにかしらのバグがあるのか実行後に「/dev/ttyACM0」が出現せず、プログラムが正常に動作しなかった。

## FeatherM4のピン定義
```

const (
	D0	= PB17	// UART0 RX/PWM available
	D1	= PB16	// UART0 TX/PWM available
	D4	= PA14	// PWM available
	D5	= PA16	// PWM available
	D6	= PA18	// PWM available
	D8	= PB03	// built-in neopixel
	D9	= PA19	// PWM available
	D10	= PA20	// can be used for PWM or UART1 TX
	D11	= PA21	// can be used for PWM or UART1 RX
	D12	= PA22	// PWM available
	D13	= PA23	// PWM available
	D21	= PA13	// PWM available
	D22	= PA12	// PWM available
	D23	= PB22	// PWM available
	D24	= PB23	// PWM available
	D25	= PA17	// PWM available
)

const (
	A0	= PA02	// ADC/AIN[0]
	A1	= PA05	// ADC/AIN[2]
	A2	= PB08	// ADC/AIN[3]
	A3	= PB09	// ADC/AIN[4]
	A4	= PA04	// ADC/AIN[5]
	A5	= PA06	// ADC/AIN[10]
)

const (
	LED = D13
)

const (
	UART_TX_PIN	= D1
	UART_RX_PIN	= D0
)

const (
	UART2_TX_PIN	= A4
	UART2_RX_PIN	= A5
)

const (
	SDA_PIN	= D22	// SDA: SERCOM2/PAD[0]
	SCL_PIN	= D21	// SCL: SERCOM2/PAD[1]
)

const (
	SPI0_SCK_PIN	= D25	// SCK: SERCOM1/PAD[1]
	SPI0_MOSI_PIN	= D24	// MOSI: SERCOM1/PAD[3]
	SPI0_MISO_PIN	= D23	// MISO: SERCOM1/PAD[2]
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

[Adafruit Feather M4 Express - Overview](https://learn.adafruit.com/adafruit-feather-m4-express-atsamd51)  
[Adafruit Feather M4 Express - PINOUT](https://learn.adafruit.com/adafruit-feather-m4-express-atsamd51/pinouts)   
[Adafruit Feater M4 Express　Pin Assign](https://tinygo.org/microcontrollers/machine/feather-m4/)  

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

