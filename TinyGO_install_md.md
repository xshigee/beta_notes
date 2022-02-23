2020/4/17  
printlnの出力先の明確化、build&flashスクリプトの追加  
2020/4/16  
自分のプログラムを置く作業ディレクトリを設定するためのGOPATH設定方法の追加  
2020/4/15  
avrdudeの説明の追加  
2020/4/13  
初版  

TinyGO Install
# TinyGO Install

## 概要
コンピュータボードでTinyGOを動かす。ここでは、対象とするボードは以下のものとする。   
F4DISCO、Arduino-Uno,Arduino-Nano,micro:bit  
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

# インストール確認
st-flash --version
#以下のように出力されればＯＫ(数字などはバージョンによって異なる)
v1.6.0-154-g99a8aaa
```

(2)openocd
```bash

sudo apt-get install openocd

```

(3)arduino関係
```bash

sudo apt-get install gcc-avr
sudo apt-get install avr-libc
sudo apt-get install avrdude

```

(4)GO  
tinyGOを動かすのにGOが必要なので
予めインストールする。
(ただし、TinyGOとの整合性により最新版ではない)
```bash

wget https://dl.google.com/go/go1.13.7.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.13.7.linux-amd64.tar.gz

export PATH=$PATH:/usr/local/go/bin
```
以上のexportは、.bashrcに登録する。

## TinyGOのインストール
```bash

mkdir tinygo_ws
cd tinygo_ws

wget https://github.com/tinygo-org/tinygo/releases/download/v0.12.0/tinygo_0.12.0_amd64.deb
sudo dpkg -i tinygo_0.12.0_amd64.deb

export PATH=$PATH:/usr/local/tinygo/bin
```

## サンプルを動かす(F4DISCO)
以下を実行することでビルドと書き込みが行わるので
ボードをホストＰＣに接続しておく。  
sample1:
```bash

tinygo flash -target stm32f4disco examples/blinky1 
```

出力ログ:
```bash
Open On-Chip Debugger 0.10.0
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
adapter speed: 2000 kHz
adapter_nsrst_delay: 100
none separate
Info : Unable to match requested speed 2000 kHz, using 1800 kHz
Info : Unable to match requested speed 2000 kHz, using 1800 kHz
Info : clock speed 1800 kHz
Info : STLINK v2 JTAG v14 API v2 SWIM v0 VID 0x0483 PID 0x3748
Info : using stlink api v2
Info : Target voltage: 2.871002
Info : stm32f4x.cpu: hardware has 6 breakpoints, 4 watchpoints
adapter speed: 2000 kHz
target halted due to debug-request, current mode: Thread 
xPSR: 0x01000000 pc: 0x08041228 msp: 0x20020000
adapter speed: 8000 kHz
** Programming Started **
auto erase enabled
Info : device id = 0x10016413
Info : flash size = 1024kbytes
target halted due to breakpoint, current mode: Thread 
xPSR: 0x61000000 pc: 0x20000046 msp: 0x20020000
wrote 16384 bytes from file /tmp/tinygo552013885/main.hex in 0.679854s (23.534 KiB/s)
** Programming Finished **
** Resetting Target **
adapter speed: 2000 kHz
shutdown command invoked
```
書き込み後、ボード上のLEDが１個点滅する。

プログラムの内容：  
/usr/local/tinygo/src/examples/blinky1/blinky1.go
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
		time.Sleep(time.Millisecond * 500)

		led.High()
		time.Sleep(time.Millisecond * 500)
	}
}
```

sample2:
```bash

tinygo flash -target stm32f4disco examples/blinky2
```

出力ログ：  
省略

書き込み後、２つのLEDが異なった周期で点滅する。

プログラムの内容：  
/usr/local/tinygo/src/examples/blinky2/blinky2.go
```go

package main

// This blinky is a bit more advanced than blink1, with two goroutines running
// at the same time and blinking a different LED. The delay of led2 is slightly
// less than half of led1, which would be hard to do without some sort of
// concurrency.

import (
	"machine"
	"time"
)

func main() {
	go led1()
	led2()
}

func led1() {
	led := machine.LED1
	led.Configure(machine.PinConfig{Mode: machine.PinOutput})
	for {
		println("+")
		led.Low()
		time.Sleep(time.Millisecond * 1000)

		println("-")
		led.High()
		time.Sleep(time.Millisecond * 1000)
	}
}

func led2() {
	led := machine.LED2
	led.Configure(machine.PinConfig{Mode: machine.PinOutput})
	for {
		println("  +")
		led.Low()
		time.Sleep(time.Millisecond * 420)

		println("  -")
		led.High()
		time.Sleep(time.Millisecond * 420)
	}
}
```

## サンプルを動かす(micro:bit)

```bash

tinygo flash -target=microbit examples/microbit-blink

```
書き込み後、micro:bitのLEDが一つ点滅する。

プログラムの内容：   
/usr/local/tinygo/src/examples/microbit-blink/microbit-blink.go 
```go


// blink program for the BBC micro:bit
package main

import (
	"machine"
	"time"
)

// The LED matrix in the micro:bit is a multiplexed display: https://en.wikipedia.org/wiki/Multiplexed_display
// Driver for easier control: https://github.com/tinygo-org/drivers/tree/master/microbitmatrix
func main() {
	ledrow := machine.LED_ROW_1
	ledrow.Configure(machine.PinConfig{Mode: machine.PinOutput})
	ledcol := machine.LED_COL_1
	ledcol.Configure(machine.PinConfig{Mode: machine.PinOutput})
	ledcol.Low()
	for {
		ledrow.Low()
		time.Sleep(time.Millisecond * 500)

		ledrow.High()
		time.Sleep(time.Millisecond * 500)
	}
}
```

## サンプルを動かす(Arduino-nano)
```bash

tinygo flash -port /dev/ttyUSB0 -target=arduino-nano examples/blinky1

```
出力ログ：
```bash

avrdude: AVR device initialized and ready to accept instructions

Reading | ################################################## | 100% 0.01s

avrdude: Device signature = 0x1e950f (probably m328p)
avrdude: NOTE: "flash" memory has been specified, an erase cycle will be performed
         To disable this feature, specify the -D option.
avrdude: erasing chip
avrdude: reading input file "/tmp/tinygo714825630/main.hex"
avrdude: writing flash (552 bytes):

Writing | ################################################## | 100% 0.32s

avrdude: 552 bytes of flash written
avrdude: verifying flash memory against /tmp/tinygo714825630/main.hex:
avrdude: load data flash data from input file /tmp/tinygo714825630/main.hex:
avrdude: input file /tmp/tinygo714825630/main.hex contains 552 bytes
avrdude: reading on-chip flash data:

Reading | ################################################## | 100% 0.27s

avrdude: verifying ...
avrdude: 552 bytes of flash verified

avrdude done.  Thank you.
```
## サンプルを動かす(Arduino-Uno)
```bash

tinygo flash -port /dev/ttyACM0 -target=arduino examples/blinky1

```
出力ログ：
```bash

avrdude: AVR device initialized and ready to accept instructions

Reading | ################################################## | 100% 0.00s

avrdude: Device signature = 0x1e950f (probably m328p)
avrdude: NOTE: "flash" memory has been specified, an erase cycle will be performed
         To disable this feature, specify the -D option.
avrdude: erasing chip
avrdude: reading input file "/tmp/tinygo194433174/main.hex"
avrdude: writing flash (552 bytes):

Writing | ################################################## | 100% 0.10s

avrdude: 552 bytes of flash written
avrdude: verifying flash memory against /tmp/tinygo194433174/main.hex:
avrdude: load data flash data from input file /tmp/tinygo194433174/main.hex:
avrdude: input file /tmp/tinygo194433174/main.hex contains 552 bytes
avrdude: reading on-chip flash data:

Reading | ################################################## | 100% 0.08s

avrdude: verifying ...
avrdude: 552 bytes of flash verified

avrdude done.  Thank you.

```

## サンプルを動かす(Seeeduino)
```bash

tinygo flash -port /dev/ttyUSB0 -target=arduino-nano examples/blinky1

```

## 自分のプログラム mycode.go を動かす
自分のプログラムを動かす場合、以下のように設定/実行する：
```bash

# GO/TinyGOの作業ディレクトリのパス設定
export GOPATH=$HOME/tinygo_ws

# プロジェクトmycodeのディレクトリ作成
mkdir -p $GOPATH/src/mycode
cd $GOPATH/src/mycode
nano mycode.go
#mycode.goとして自分のプログラムを書く

# F4DISCOで動かす場合は、以下のようにする：
tinygo flash -target stm32f4disco mycode

```


## dockerで TinyGO を動かす
上で述べたTinyGOのインストールがライブラリの問題で正しくインストールできないことがある。
その場合は、dockerを使用してTinyGOを動かすことができるので、その方法について記する。   
(1)dockerのインストール
```bash

sudo apt-get docker
sudo apt-get docker.io

sudo groupadd docker 
sudo usermod -aG docker $USER
sudo systemctl enable docker
sudo systemctl start docker
#ここで再起動する
```
(2)tinygoのインストール
```bash

docker pull tinygo/tinygo:0.12.0
```

## docker/tinygoによるビルド(F4DISCO)
```bash

docker run --rm -v $(pwd):/src -v $GOPATH:/go -e "GOPATH=/go" tinygo/tinygo:0.12.0 tinygo build -o /src/F4D_blinky1.bin -target=stm32f4disco examples/blinky1

```
これで、カレントディレクトリにバイナリF4D_blinky1.binが生成される。

## st-flashによる書き込み(F4DISCO)
```bash

st-flash write F4D_blinky1.bin 0x8000000
```

以上が一連の手続きになるが、コマンドが長過ぎるので
以下のようにaliasを使うとコマンドが短くなる。
```bash

alias tigo='docker run --rm -v $(pwd):/src -v $GOPATH:/go -e "GOPATH=/go" tinygo/tinygo:0.12.0 tinygo '
tigo build -o /src/F4D_blinky2.bin -target=stm32f4disco examples/blinky2

```

## docker/tinygoによるビルド(micro:bit)
```bash

tigo build -o /src/mb.hex -target=microbit examples/microbit-blink
```
mb.hexができるので、それをmicro:bitのストレージにドラッグ＆ドロップして書き込む。自分のmycodeをビルドする場合、「examples/microbit-blink」を「mycode」に変更して実行する。

## docker/tinygoによるビルド(Arduino-nano)
```bash

tigo build  -o /src/nano.hex -target=arduino-nano examples/blinky1
```
nano.hexが生成されるのでavrdudeで書き込む。
自分のmycodeをビルドする場合、「examples/blinky1」を「mycode」に変更して実行する。

## docker/tinygoによるビルド(Arduino-UNO)
```bash

tigo build -o /src/uno.hex -target=arduino examples/blinky1
```
uno.hexが生成されるので、avrdudeで書き込む。
自分のmycodeをビルドする場合、「examples/blinky1」を「mycode」に変更して実行する。

## avrduinoを使った書き込み
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
補足：   
「pio run -t nobuild -t upload -v」を実行すると
実際に実行されるコマンドラインを出力するので
別のボードを使うときの書き込み方法を知ることができる。

## .bashrc　に追加するコマンド(まとめ)
.bashrcの末尾に以下を追加する：
```bash

# GO/TinyGO
export PATH=$PATH:/usr/local/go/bin
export PATH=$PATH:/usr/local/tinygo/bin
export GOPATH=$HOME/tinygo_ws
alias tigo='docker run --rm -v $(pwd):/src -v $GOPATH:/go -e "GOPATH=/go" tinygo/tinygo:0.12.0 tinygo '

```

## build&flashスクリプト
書き込みを含めると、まだ、入力が長いので、スクリプトにする。

micro:bit用  
fmb.sh
```bash

#!/bin/bash
docker run --rm -v $(pwd):/src -v $GOPATH:/go -e "GOPATH=/go" tinygo/tinygo:0.12.0 tinygo build -o /src/mb.hex -target=microbit $1
cp mb.hex /media/USER_NAME/MICROBIT/
```
「USER_NAME」の部分は、実際の環境に合わせて変更すること。

nano用  
fnano.sh
```bash

#!/bin/bash
docker run --rm -v $(pwd):/src -v $GOPATH:/go -e "GOPATH=/go" tinygo/tinygo:0.12.0 tinygo build  -o /src/nano.hex -target=arduino-nano $1
avrdude -p atmega328p -C ~/.platformio/packages/tool-avrdude/avrdude.conf -c arduino -b 57600 -D -P /dev/ttyUSB0 -U flash:w:nano.hex:i
```
uno用  
funo.sh
```bash

#!/bin/bash
docker run --rm -v $(pwd):/src -v $GOPATH:/go -e "GOPATH=/go" tinygo/tinygo:0.12.0 tinygo build  -o /src/uno.hex -target=arduino $1
avrdude -p atmega328p -C ~/.platformio/packages/tool-avrdude/avrdude.conf -c arduino -b 115200 -D -P /dev/ttyACM0 -U flash:w:uno.hex:i
```

実行属性を与える必要があるので
「chmod +x xxx.sh」を実行して実行属性を与えること。

実行例：
```bash

# ターゲットがmicro:bitの場合
./fmb.sh hello

# ターゲットがnanoの場合
./fnano.sh hello

# ターゲットがunoの場合
./funo.sh hello

```

## printlnの出力先
ボードによって実装が多少異なっているようで
F4DISCO,nanoは、ボードの(シリアル)ピンに出力される。
unoとmicro:bitは、プログラミングで使用しているUSBに
出力されるが、ビットレートは異なる。
以下、通信ソフトpicocomで接続した例をを示す：
```bash

# nanoの場合
# シリアル用ピンをMCP2221AなどUSBserialボード経由で接続する
picocom /dev/ttyACM0 -b9600 

# unoの場合
# ボードのUSBserial(プログラム書き込み用と兼用)に接続する
picocom /dev/ttyACM0 -b19200

# micro:bitの場合
# ボードのUSBserialに接続する
picocom /dev/ttyACM0 -b115200

```

シリアル出力テスト用プログラム
src/hello/hello.go
```go

package main

func main() {
     // setup
     println("Start Seriral.")

     // loop
     for{
        println("Hello TinyGO World!")
     }
}
```

実行例：
```bash

# unoの場合
./funo.sh hello
picocom /dev/ttyACM0 -b19200

# micro:bitの場合
./fmb.sh hello
picocom /dev/ttyACM0 -b115200

```

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

