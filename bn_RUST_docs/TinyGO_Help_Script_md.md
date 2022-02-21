
2020/4/23  
build用スクリプト追加  
2020/4/20+  
初版  

docker/TinyGO Helper Script
# docker/TinyGO Helper Script

## 概要
docker/TinyGOでビルドや書き込みを行なう場合、
そのままでは、コマンドラインが長くなり使いにくい。
それを軽減するためにbashのスクリプトを用意した。
(ホストPCとしてはubuntuを想定している)

## 準備
以下のツールを予めインストールする：    
(1)Docker/TinyGOを最新版にする
```bash

docker pull tinygo/tinygo:latest

``` 

(2)arduino関係
```bash

sudo apt-get install gcc-avr
sudo apt-get install avr-libc
sudo apt-get install avrdude
```

(3)PlatformIO  
書き込みツールでplatformioのavrdude.confを参照しているので、platformioで
arduinoのプログラムのビルドと書き込みを行なうこと。（これをやることで、自動的にavrdude.confがダウンロードされる）

(4)st-flash  
STM32用書き込みツール(st-flash)をインストールする：
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

## Arduino-UNO用
fsunoC.sh 
```bash
#!/bin/bash
rm -f uno.hex
docker run -it --rm -v $PWD:/go/src/github.com/myusr/myapp -w /go/src/github.com/myusr/myapp -e GOPATH=/go tinygo/tinygo:latest tinygo build -size full -o uno.hex -target arduino .
if test -f ./uno.hex; then
  avrdude -p atmega328p -C ~/.platformio/packages/tool-avrdude/avrdude.conf -c arduino -b 115200 -D -P /dev/ttyACM0 -U flash:w:uno.hex:i
  picocom /dev/ttyACM0 -b19200
fi
```
プログラム(TinyGO)のなかのprintlnに対応するために、プログラム書き込み後、自動的にpicocomを起動している。

実行例：   
プログラムの置いてあるディレクトリで以下を実行する：
```bash

./fsunoC.sh

   code  rodata    data     bss |   flash     ram | package
    132     152       0       0 |     284       0 | (bootstrap)
    446       0       3       0 |     449       3 | github
     74       0       0       0 |      74       0 | machine
      0       0       0     130 |       0     130 | machine$alloc
    216       0      53      10 |     269      63 | runtime
    868     152      56     140 |    1076     196 | (sum)
   1192       -      73     658 |    1265     731 | (all)

avrdude: AVR device initialized and ready to accept instructions

Reading | ################################################## | 100% 0.00s

avrdude: Device signature = 0x1e950f (probably m328p)
avrdude: reading input file "uno.hex"
avrdude: writing flash (1266 bytes):

Writing | ################################################## | 100% 0.20s

avrdude: 1266 bytes of flash written
avrdude: verifying flash memory against uno.hex:
avrdude: load data flash data from input file uno.hex:
avrdude: input file uno.hex contains 1266 bytes
avrdude: reading on-chip flash data:

Reading | ################################################## | 100% 0.16s

avrdude: verifying ...
avrdude: 1266 bytes of flash verified

avrdude: safemode: Fuses OK (E:00, H:00, L:00)

avrdude done.  Thank you.

picocom v1.7

port is        : /dev/ttyACM0
flowcontrol    : none
baudrate is    : 19200
parity is      : none
databits are   : 8
escape is      : C-a
local echo is  : no
noinit is      : no
noreset is     : no
nolock is      : no
send_cmd is    : sz -vv
receive_cmd is : rz -vv
imap is        : 
omap is        : 
emap is        : crcrlf,delbs,

Terminal ready
A4: 7936
A4: 8192
A4: 8768
A4: 9344
A4: 9664
A4: 9792
.....

```

## Arduino-nano用
fsnanoC.sh
```bash

#!/bin/bash
rm -f nano.hex
docker run -it --rm -v $PWD:/go/src/github.com/myusr/myapp -w /go/src/github.com/myusr/myapp -e GOPATH=/go tinygo/tinygo:latest tinygo build -size full -o nano.hex -target arduino-nano .
if test -f ./nano.hex; then
  avrdude -p atmega328p -C ~/.platformio/packages/tool-avrdude/avrdude.conf -c arduino -b 57600 -D -P /dev/ttyUSB0 -U flash:w:nano.hex:i
  picocom /dev/ttyUSB0 -b9600
fi
```
プログラム(TinyGO)のなかのprintlnに対応するために、プログラム書き込み後、自動的にpicocomを起動している。

実行例：  
プログラムの置いてあるディレクトリで以下を実行する：
```bash

./fsnanoC.sh 

   code  rodata    data     bss |   flash     ram | package
    132     152       0       0 |     284       0 | (bootstrap)
    446       0       3       0 |     449       3 | github
     74       0       0       0 |      74       0 | machine
      0       0       0     130 |       0     130 | machine$alloc
    216       0      53      10 |     269      63 | runtime
    868     152      56     140 |    1076     196 | (sum)
   1192       -      73     658 |    1265     731 | (all)

avrdude: AVR device initialized and ready to accept instructions

Reading | ################################################## | 100% 0.01s

<省略>

avrdude done.  Thank you.

picocom v1.7

port is        : /dev/ttyUSB0
flowcontrol    : none
baudrate is    : 9600
parity is      : none
databits are   : 8
escape is      : C-a
local echo is  : no
noinit is      : no
noreset is     : no
nolock is      : no
send_cmd is    : sz -vv
receive_cmd is : rz -vv
imap is        : 
omap is        : 
emap is        : crcrlf,delbs,

Terminal ready
A4: 46016
A4: 46144
A4: 46336
A4: 46272
A4: 46336


```

## Seeeduino用
nano互換だがシリアルデバイス名や指定するビットレートなどが異なる。  
fsseedC.sh
```bash

#!/bin/bash
rm -f nano.hex
docker run -it --rm -v $PWD:/go/src/github.com/myusr/myapp -w /go/src/github.com/myusr/myapp -e GOPATH=/go tinygo/tinygo:latest tinygo build -size full -o nano.hex -target arduino-nano .
if test -f ./nano.hex; then
  avrdude -p atmega328p -C ~/.platformio/packages/tool-avrdude/avrdude.conf -c arduino -b 57600 -D -P /dev/ttyUSB0 -U flash:w:nano.hex:i
  picocom /dev/ttyUSB0 -b9600
fi
```
プログラム(TinyGO)のなかのprintlnに対応するために、プログラム書き込み後、自動的にpicocomを起動している。

実行例：   
プログラムの置いてあるディレクトリで以下を実行する：
```bash

./fsseedC.sh 

   code  rodata    data     bss |   flash     ram | package
    132     152       0       0 |     284       0 | (bootstrap)
    446       0       3       0 |     449       3 | github
     74       0       0       0 |      74       0 | machine
      0       0       0     130 |       0     130 | machine$alloc
    216       0      53      10 |     269      63 | runtime
    868     152      56     140 |    1076     196 | (sum)
   1192       -      73     658 |    1265     731 | (all)

avrdude: AVR device initialized and ready to accept instructions

Reading | ################################################## | 100% 0.01s

＜省略＞

avrdude done.  Thank you.

picocom v1.7

port is        : /dev/ttyUSB0
flowcontrol    : none
baudrate is    : 9600
parity is      : none
databits are   : 8
escape is      : C-a
local echo is  : no
noinit is      : no
noreset is     : no
nolock is      : no
send_cmd is    : sz -vv
receive_cmd is : rz -vv
imap is        : 
omap is        : 
emap is        : crcrlf,delbs,

Terminal ready
A4: 22784
A4: 48896
A4: 29120
A4: 57728
A4: 28736
A4: 61952


```

## micro:bit用
fsmbC.sh
```bash

#!/bin/bash
rm -f mb.hex
docker run -it --rm -v $PWD:/go/src/github.com/myusr/myapp -w /go/src/github.com/myusr/myapp -e GOPATH=/go tinygo/tinygo:latest tinygo build -size full -o mb.hex -target microbit .
if test -f ./mb.hex; then
  cp mb.hex /media/USER_NAME/MICROBIT/
  picocom /dev/ttyACM0 -b115200
fi
```
USER_NAMEは、自分の環境に合わせて変更すること。   
プログラム(TinyGO)のなかのprintlnに対応するために、プログラム書き込み後、自動的にpicocomを起動している。

実行例：  
プログラムの置いてあるディレクトリで以下を実行する：
```bash

./fsmbC.sh

   code  rodata    data     bss |   flash     ram | package
   1082      13       0       0 |    1095       0 | (bootstrap)
    324      16       0       0 |     340       0 | github
      0      36       0       0 |      36       0 | handleHardFault$string
    108      24       0       4 |     132       4 | internal/task
    234       0       0       0 |     234       0 | machine
      0       0       0     130 |       0     130 | machine$alloc
   1586      79       0      41 |    1665      41 | runtime
   3334     168       0     175 |    3502     175 | (sum)
   3820       -       0    2244 |    3820    2244 | (all)
picocom v1.7

port is        : /dev/ttyACM0
flowcontrol    : none
baudrate is    : 115200
parity is      : none
databits are   : 8
escape is      : C-a
local echo is  : no
noinit is      : no
noreset is     : no
nolock is      : no
send_cmd is    : sz -vv
receive_cmd is : rz -vv
imap is        : 
omap is        : 
emap is        : crcrlf,delbs,

Terminal ready
Program Start...
...
...

```
プログラム(TinyGO)のなかのprintlnに対応するために、プログラム書き込み後、自動的にpicocomを起動している。

## F4DISCO用
fsf4dC.sh 
```bash

#!/bin/bash
rm -f f4d.bin
docker run -it --rm -v $PWD:/go/src/github.com/myusr/myapp -w /go/src/github.com/myusr/myapp -e GOPATH=/go tinygo/tinygo:latest tinygo build -size full -o f4d.bin -target stm32f4disco .
if test -f ./f4d.bin; then
  st-flash write f4d.bin 0x8000000
  picocom /dev/ttyACM0 -b115200
fi
```
picocomのデバイス名は、使用しているUSBserialに合わせること。
プログラム(TinyGO)のなかのprintlnに対応するために、プログラム書き込み後、自動的にpicocomを起動している。  

USBserialとF4DISCOボードの接続図：
| USBserial | F4DISCO |
| :--: | :--: |
| GND | GND |
| TXD | PA3(USART2_RX)|
| RXD | PA2(USART2_TX)|

実行例：  
プログラムの置いてあるディレクトリで以下を実行する：
```bash

./fsf4dC.sh

   code  rodata    data     bss |   flash     ram | package
    628      13       0     130 |     641     130 | (bootstrap)
     36      33       0       0 |      69       0 | github
      0      36       0       0 |      36       0 | handleHardFault$string
      0      24       0       4 |      24       4 | internal/task
   1752      79       0      25 |    1831      25 | runtime
     26       0       0       0 |      26       0 | runtime/volatile
   2442     185       0     159 |    2627     159 | (sum)
   3192       -       0    4276 |    3192    4276 | (all)
st-flash 1.6.0
2020-04-20T21:49:18 INFO common.c: Loading device parameters....
2020-04-20T21:49:18 INFO common.c: Device connected is: F4 device, id 0x10016413
2020-04-20T21:49:18 INFO common.c: SRAM size: 0x30000 bytes (192 KiB), Flash: 0x100000 bytes (1024 KiB) in pages of 16384 bytes
2020-04-20T21:49:18 INFO common.c: Attempting to write 3192 (0xc78) bytes to stm32 address: 134217728 (0x8000000)
Flash page at addr: 0x08000000 erased
2020-04-20T21:49:19 INFO common.c: Finished erasing 1 pages of 16384 (0x4000) bytes
2020-04-20T21:49:19 INFO common.c: Starting Flash write for F2/F4/L4
2020-04-20T21:49:19 INFO flash_loader.c: Successfully loaded flash loader in sram
enabling 32-bit flash writes
size: 3192
2020-04-20T21:49:19 INFO common.c: Starting verification of write complete
2020-04-20T21:49:19 INFO common.c: Flash written and verified! jolly good!
picocom v1.7

port is        : /dev/ttyACM0
flowcontrol    : none
baudrate is    : 115200
parity is      : none
databits are   : 8
escape is      : C-a
local echo is  : no
noinit is      : no
noreset is     : no
nolock is      : no
send_cmd is    : sz -vv
receive_cmd is : rz -vv
imap is        : 
omap is        : 
emap is        : crcrlf,delbs,

Terminal ready
Hello TinyGO World!
Hello TinyGO World!
Hello TinyGO World!
Hello TinyGO World!
...
...

```
なお、スクリプトに実行属性を与えるには以下を実行する：
```bash

chmod +x xxxx.sh

```

## build用スクリプト
uf2出力用  
uf2_build.sh 
```bash

#!/bin/bash
docker run -it --rm -v $PWD:/go/src/github.com/myusr/myapp -w /go/src/github.com/myusr/myapp -e GOPATH=/go tinygo/tinygo:latest tinygo build -size full -o out.uf2 -target $1 .
```

hex出力用  
hex_build.sh
```bash

#!/bin/bash
docker run -it --rm -v $PWD:/go/src/github.com/myusr/myapp -w /go/src/github.com/myusr/myapp -e GOPATH=/go tinygo/tinygo:latest tinygo build -size full -o out.hex -target $1 .
```

bin出力用  
bin_build.sh
```bash

#!/bin/bash
docker run -it --rm -v $PWD:/go/src/github.com/myusr/myapp -w /go/src/github.com/myusr/myapp -e GOPATH=/go tinygo/tinygo:latest tinygo build -size full -o out.bin -target $1 .
```

実行例：
```bash

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

cd $GOPATH/src
#　ここでは例として「lis3dh」をインポートする
mkdir lis3dh
cd lis3dh

# モジュールをディレクトリの形でコピーする
cp -r $TIGOLIBS/lis3dh .

# exampleのmain.goをコピーする
cp $TIGOLIBS/examples/lis3dh/main.go .

# main.goを編集する
leafpad main.go
# 以下のように修正する：
# "tinygo.org/x/drivers/lis3dh"　→　"./lis3dh"
# dockerを実行環境にする場合、インポートモジュールをカレント・ディレクトリに置く必要がある

# ファイルの構成のチェックのため circuitplay-express でビルドする
./uf2_build.sh circuitplay-express
以下のようなマップが出力されればビルドとしてはＯＫ：
   code  rodata    data     bss |   flash     ram | package
   4372      13       0       0 |    4385       0 | (bootstrap)
     18       0       0       0 |      18       0 | errors
    732      30       0       0 |     762       0 | github
      0      36       0       0 |      36       0 | handleHardFault$string
    108      24       0       4 |     132       4 | internal/task
   3274     138       5    2054 |    3417    2059 | machine
      0      16      32     130 |      48     162 | machine$alloc
   2396     106       0      45 |    2502      45 | runtime
  10900     363      37    2233 |   11300    2270 | (sum)
  11624       -      40    4312 |   11664    4352 | (all)
#　circuitplay-expressがボードとして実装が進んでいるのでチェック用ターゲットとして採用した

# 以下、実際のターゲット用にビルドする
# 実装が進んでいないボードでは、エラーが出てビルドできないはず。

# 以下、ビルドが上手く言った例：(ピンアサインなどをボードに合わせて修正してある)
$ ./bin_build.sh nucleo-f103rb
   code  rodata    data     bss |   flash     ram | package
   1644      21       0     130 |    1665     130 | (bootstrap)
     16       0       0       0 |      16       0 | errors
    824      30       0       0 |     854       0 | github
      0      36       0       0 |      36       0 | handleHardFault$string
    102      24       0       4 |     126       4 | internal/task
   1298     121       0       0 |    1419       0 | machine
      0       0      32       0 |      32      32 | machine$alloc
   1804     113       0      45 |    1917      45 | runtime
     38       0       0       0 |      38       0 | runtime/volatile
   5726     345      32     179 |    6103     211 | (sum)
   6536       -      32    2244 |    6568    2276 | (all)

# 以下、エラーが出る例：
$ ./hex_build.sh arduino
# github.com/myusr/myapp
main.go:12:19: I2C1 not declared by package machine
main.go:15:47: SCL1_PIN not declared by package machine
main.go:15:34: unknown field SCL in struct literal
main.go:15:70: SDA1_PIN not declared by package machine
main.go:15:57: unknown field SDA in struct literal
# 修正方法があるのかもしれないが、今のところ不明

```

## ボードのターゲット名
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

[TinyGoで始める組み込みプログラミング](https://tech.144lab.com/entry/tinygo)
[TinyGo on Arduino Uno: An Introduction](https://create.arduino.cc/projecthub/alankrantas/tinygo-on-arduino-uno-an-introduction-6130f6)  

[STM32F4DISCO　Pin Assign](https://tinygo.org/microcontrollers/machine/stm32f4disco/)  
[MICROBIT Pin Assign](https://tinygo.org/microcontrollers/machine/microbit/)  
[ARDUINO-NANO Pin Assign](https://tinygo.org/microcontrollers/machine/arduino-nano/)   
[ARDUINO Pin Assign](https://tinygo.org/microcontrollers/machine/arduino/)  

[TinyGo Drivers](https://github.com/tinygo-org/drivers/)  


以上
