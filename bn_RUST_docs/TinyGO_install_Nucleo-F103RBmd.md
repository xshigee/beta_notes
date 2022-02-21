
2020/5/26:  
分かりやすくするために一部修正した。

2020/5/12:  
初版  

TinyGO Install Nucleo-F103RB
# TinyGO Install Nucleo-F103RB

## 概要
Nucleo-F103RBボードでTinyGOを動かす。
ここでは、Docker版のTinyGOを使用する。
(ホストPCとしてはubuntuを想定している)

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

(4)GOのインストール  
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


(5)alias用スクリプトの設定  
nano ~/setenv4tg   
以下の内容に編集する：
```

# for nucleo-f103rb
alias tgf103="docker run -it --rm -v $PWD:/go/src/github.com/myusr/myapp -w /go/src/github.com/myusr/myapp -e GOPATH=/go tinygo/tinygo:latest tinygo build -size full -o firmware.bin -target nucleo-f103rb ."
alias stf="st-flash write firmware.bin 0x8000000"
```

以上のexportは、.bashrcに登録する。


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

tgf103

出力例：
```bash
docker run -it --rm -v $PWD:/go/src/github.com/myusr/myapp -w /go/src/github.com/myusr/myapp -e GOPATH=/go tinygo/tinygo:latest tinygo build -size full -o firmware.bin -target nucleo-f103rb .
   code  rodata    data     bss |   flash     ram | package
    796      21       0     130 |     817     130 | (bootstrap)
     38       0       0       0 |      38       0 | github
      0      36       0       0 |      36       0 | handleHardFault$string
    102      24       0       4 |     126       4 | internal/task
    202      21       0       0 |     223       0 | machine
   1690      86       0      45 |    1776      45 | runtime
     38       0       0       0 |      38       0 | runtime/volatile
    104       0       0       0 |     104       0 | time
   2970     188       0     179 |    3158     179 | (sum)
   3596       -       0    2244 |    3596    2244 | (all)


```

(4)書き込み   
以下を実行して書き込む：
```bash

#ボードとホストPCをUSBで接続する:
stf

```
ボードのグリーンLEDが１秒周期で点滅すれば動作としてはＯＫとなる。

出力例：
```bash

$ stf
st-flash 1.6.0
2020-05-12T11:59:36 INFO common.c: Loading device parameters....
2020-05-12T11:59:36 INFO common.c: Device connected is: F1 Medium-density device, id 0x20036410
2020-05-12T11:59:36 INFO common.c: SRAM size: 0x5000 bytes (20 KiB), Flash: 0x20000 bytes (128 KiB) in pages of 1024 bytes
2020-05-12T11:59:36 INFO common.c: Attempting to write 3596 (0xe0c) bytes to stm32 address: 134217728 (0x8000000)
Flash page at addr: 0x08000c00 erased
2020-05-12T11:59:37 INFO common.c: Finished erasing 4 pages of 1024 (0x400) bytes
2020-05-12T11:59:37 INFO common.c: Starting Flash write for VL/F0/F3/F1_XL core id
2020-05-12T11:59:37 INFO flash_loader.c: Successfully loaded flash loader in sram
  4/4 pages written
2020-05-12T11:59:37 INFO common.c: Starting verification of write complete
2020-05-12T11:59:37 INFO common.c: Flash written and verified! jolly good!
```


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

(3)ビルド/書き込み  
以下を実行する：
```bash

. ~/setenv4tg
tgf103
stf

```
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

#　ここでは例として「tmp102」をインポートする
mkdir tmp102
cd tmp102

# モジュールをディレクトリの形でコピーする
cp -r $TIGOLIBS/tmp102 .

# exampleのmain.goをコピーする
cp $TIGOLIBS/examples/tmp102/main.go .

# main.goを編集する
leafpad main.go
# 以下のように修正する：
# "tinygo.org/x/drivers/tmp102" →  "./tmp102"
# dockerを実行環境にする場合、インポートモジュールをカレント・ディレクトリに置く必要がある

# 以下、ビルド&書き込み
. ~/setenv4tg
tgf103
stf

```
以上で、例としてtmp102のプログラムがビルド＆実行できる。  
注意：I2Cの接続が、Arduinoのピン配列と異なり
「[NUCLEO-F103RB Pin Assign](https://tinygo.org/microcontrollers/machine/nucleo-f103rb/)」のピン配置になるので
以下になる：  
```
  
SCL_PIN	= PB6(D10)   
SDA_PIN	= PB7(CN7上)　# CN6のマーキングの２つあるGNDの下の方の位置  
```  
正常動作であればシリアル出力で気温が出力される。

出力例：
```bash

$ picocom /dev/ttyACM0 -b115200
...

Terminal ready
27.12°C
27.12°C
...
...
27.12°C
27.19°C
27.25°C
27.38°C
27.56°C
28.06°C
30.12°C # ←動作確認のためにセンサーを指で触って温度を上げた 
..

```

## NUCLEO-F103RBのピン定義

```

const (
	LED		= LED_BUILTIN
	LED_BUILTIN	= LED_GREEN
	LED_GREEN	= PA5
)

const (
	UART_TX_PIN	= PA2
	UART_RX_PIN	= PA3
	UART_ALT_TX_PIN	= PD5
	UART_ALT_RX_PIN	= PD6
)

const (
	SCL_PIN	= PB6
	SDA_PIN	= PB7
)

const (
	SPI0_SCK_PIN	= PA5
	SPI0_MISO_PIN	= PA6
	SPI0_MOSI_PIN	= PA7
)

```

## ボードのターゲット名
「-target nucleo-f103rb」を以下にあるターゲット名に変更すると
他のボード用にビルドできる。  

主なもののみ(=個人的に興味があるもの)  
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

(7)Arduino Nano  
arduino-nano  

(8)Arduino Nano33 IoT  
arduino-nano33  

(9)Arduino Uno  
arduino  

(10)BBC micro:bit  
microbit  

(11)SiFIve HiFive1  
hifive1b  

(12)ST Micro "Nucleo F103RB"  
nucleo-f103rb  

(13)ST Micro STM32F103XX "Bluepill"  
bluepill  

(14)ST Micro STM32F407 "Discovery"  
stm32f4disco  
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

[NUCLEO-F103RB mbed pinout](https://os.mbed.com/platforms/ST-Nucleo-F103RB/)   
[NUCLEO-F103RB Pin Assign](https://tinygo.org/microcontrollers/machine/nucleo-f103rb/)  

[STM32F4DISCO　Pin Assign](https://tinygo.org/microcontrollers/machine/stm32f4disco/)  
[MICROBIT Pin Assign](https://tinygo.org/microcontrollers/machine/microbit/)  
[ARDUINO-NANO Pin Assign](https://tinygo.org/microcontrollers/machine/arduino-nano/)   
[ARDUINO Pin Assign](https://tinygo.org/microcontrollers/machine/arduino/)  

[TinyGo Drivers](https://github.com/tinygo-org/drivers/)  


以上

