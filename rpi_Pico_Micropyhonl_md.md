
2021/2/3

rpi Pico MicroPython/CircuitPython Install
# rpi Pico MicroPython/CircuitPython Install

## 概要
以下のPicoボードにMicropython/CircuitPythonをインストールする方法について記載する。   
(ホストPCとしてはubuntuを想定している)

・[Raspberry Pi Pico](https://www.switch-science.com/catalog/6900/)  


## MicroPytho PreBuild 
プリビルドしたファームウェアを使う場合  
以下を手順でファームウェアをダウンロードする：
```

cd ~/Downloads
wget https://www.raspberrypi.org/documentation/pico/getting-started/static/5d8e777377e8dbe23cf36360d6efc727/pico_micropython_20210121.uf2

```
このファームウェアを「ビルドしたファームウェアの書き込み」の手順で書き込む。

ソースからビルドする場合、以降の手順でビルドする。

## 事前準備
以下の手順でツールをインストールする：
```

sudo apt update
sudo apt install gcc-arm-none-eabi build-essential 

# ビルド時にエラーになったのでcmake最新版をインストールする
# 古いcmakeを削除する
sudo apt remove cmake

mkdir ~/cmake_latest
cd ~/cmake_latest

wget https://github.com/Kitware/CMake/releases/download/v3.17.1/cmake-3.17.1.tar.gz
tar zxvf cmake-3.17.1.tar.gz

cd cmake-3.17.1/
./bootstrap
make
sudo make install
```

## 該当ツールのインストール
以下の手順でインストールする：
```

# pico SDK のダウンロード
mkdir ~/pico
git clone -b master https://github.com/raspberrypi/pico-sdk.git
cd pico-sdk
git submodule update --init

# picotoolのインストール
cd ~/pico
git clone https://github.com/raspberrypi/picotool.git
cd picotool
mkdir build
cd build
export PICO_SDK_PATH=../../pico-sdk
cmake ..
make

sudo cp picotool  /usr/local/bin/

# elf2uf2のインストール
cd ~/pico
cd pico-sdk/tools/elf2uf2
mkdir build
cd build
export PICO_SDK_PATH=../../../../pico-sdk
cmake ..
make

sudo cp elf2uf2  /usr/local/bin/

# pioasmのインストール
cd ~/pico
cd pico-sdk/tools/pioasm
mkdir build
cd build
export PICO_SDK_PATH=../../../../pico-sdk
cmake ..
make

sudo cp pioasm  /usr/local/bin/
```

## MicroPython Build
以下の手順でビルドする：
```

cd ~/pico

git clone -b pico https://github.com/raspberrypi/micropython.git

cd micropython
git submodule update --init -- lib/pico-sdk
cd lib/pico-sdk
git submodule update --init

cd ../..
make -C mpy-cross
cd ports/rp2
make clean
make

ls build
CMakeCache.txt       elf2uf2       firmware.elf.map  frozen_mpy
CMakeFiles           firmware.bin  firmware.hex      generated
Makefile             firmware.dis  firmware.uf2      genhdr
cmake_install.cmake  firmware.elf  frozen_content.c  pico_sdk
```
この時点でfirmware.*がビルドされる。 
実際にPicoボードで書き込むのは、firmware.uf2になる。

## ビルドしたファームウェアの確認
以下のやり方でファームウェアの内容を確認できる：
```

$ picotool info firmware.uf2
File firmware.uf2:

Program Information
 name:            MicroPython
 version:         de1239b6a
 features:        USB REPL
                  thread support
 frozen modules:  _boot, rp2, ds18x20, onewire, uasyncio, uasyncio/core,
                  uasyncio/event, uasyncio/funcs, uasyncio/lock, uasyncio/stream
```

## ビルドしたファームウェアの書き込み
以下の手順で書き込む：
```

# BOOTSELボタンを押しながらPicoボードをホストPCにUSB接続する。
# 接続後、BOOTSELボタンを離す。

cd build
sudo cp firmwre.uf2 /media/<USER>/RPI-RP2
```
以上でfirmwareがボードに書き込まれる。   
(\<USER\>の部分はホストの環境に合わせる)

## 動作確認(REPL)
picocomを使いボードとシリアルで通信する。   
以下、通信例：
```

$ picocom /dev/ttyACM0 -b115200

MPY: soft reboot
MicroPython de1239b6a on 2021-01-24; Raspberry Pi Pico with RP2040
Type "help()" for more information.
>>> import os
>>> os.uname()
(sysname='rp2', nodename='rp2', release='1.13.0', version='de1239b6a on 2021-01-24 (GNU 6.3.1 MinSizeRel)', machine='Raspberry Pi Pico with RP2040')
>>>
>>> import gc
>>> gc.collect()
>>> gc.mem_free()
187904
```

オンライン・ヘルプ表示：
```
>>> help()
Welcome to MicroPython!

For online help please visit https://micropython.org/help/.

For access to the hardware use the 'machine' module.  RP2 specific commands
are in the 'rp2' module.

Quick overview of some objects:
  machine.Pin(pin) -- get a pin, eg machine.Pin(0)
  machine.Pin(pin, m, [p]) -- get a pin and configure it for IO mode m, pull mode p
    methods: init(..), value([v]), high(), low(), irq(handler)
  machine.ADC(pin) -- make an analog object from a pin
    methods: read_u16()
  machine.PWM(pin) -- make a PWM object from a pin
    methods: deinit(), freq([f]), duty_u16([d]), duty_ns([d])
  machine.I2C(id) -- create an I2C object (id=0,1)
    methods: readfrom(addr, buf, stop=True), writeto(addr, buf, stop=True)
             readfrom_mem(addr, memaddr, arg), writeto_mem(addr, memaddr, arg)
  machine.SPI(id, baudrate=1000000) -- create an SPI object (id=0,1)
    methods: read(nbytes, write=0x00), write(buf), write_readinto(wr_buf, rd_buf)
  machine.Timer(freq, callback) -- create a software timer object
    eg: machine.Timer(freq=1, callback=lambda t:print(t))

Pins are numbered 0-29, and 26-29 have ADC capabilities
Pin IO modes are: Pin.IN, Pin.OUT, Pin.ALT
Pin pull modes are: Pin.PULL_UP, Pin.PULL_DOWN

Useful control commands:
  CTRL-C -- interrupt a running program
  CTRL-D -- on a blank line, do a soft reset of the board
  CTRL-E -- on a blank line, enter paste mode

For further help on a specific object, type help(obj)
For a list of available modules, type help('modules')
>>> help('modules')
__main__          framebuf          uasyncio/__init__ uio
_boot             gc                uasyncio/core     uos
_onewire          machine           uasyncio/event    urandom
_rp2              math              uasyncio/funcs    uselect
_thread           micropython       uasyncio/lock     ustruct
_uasyncio         onewire           uasyncio/stream   usys
builtins          rp2               ubinascii         utime
ds18x20           uarray            ucollections
Plus any modules on the filesystem
>>>
>>>
>>>
```

## MicroPython demo scripts

blink.py
```python

from time import sleep
from machine import Pin

led = Pin(25, Pin.OUT)

while True:
    led.value(1)
    sleep(0.1)
    led.value(0)
    sleep(0.1)
```

neopixel_ring.py  
```python

# Example using PIO to drive a set of WS2812 LEDs.

import array, time
from machine import Pin
import rp2

# Configure the number of WS2812 LEDs.
NUM_LEDS = 24 #16
PIN_NUM = 6
brightness = 0.2

@rp2.asm_pio(sideset_init=rp2.PIO.OUT_LOW, out_shiftdir=rp2.PIO.SHIFT_LEFT, autopull=True, pull_thresh=24)
def ws2812():
    T1 = 2
    T2 = 5
    T3 = 3
    wrap_target()
    label("bitloop")
    out(x, 1)               .side(0)    [T3 - 1]
    jmp(not_x, "do_zero")   .side(1)    [T1 - 1]
    jmp("bitloop")          .side(1)    [T2 - 1]
    label("do_zero")
    nop()                   .side(0)    [T2 - 1]
    wrap()


# Create the StateMachine with the ws2812 program, outputting on pin
sm = rp2.StateMachine(0, ws2812, freq=8_000_000, sideset_base=Pin(PIN_NUM))

# Start the StateMachine, it will wait for data on its FIFO.
sm.active(1)

# Display a pattern on the LEDs via an array of LED RGB values.
ar = array.array("I", [0 for _ in range(NUM_LEDS)])

##########################################################################
def pixels_show():
    dimmer_ar = array.array("I", [0 for _ in range(NUM_LEDS)])
    for i,c in enumerate(ar):
        r = int(((c >> 8) & 0xFF) * brightness)
        g = int(((c >> 16) & 0xFF) * brightness)
        b = int((c & 0xFF) * brightness)
        dimmer_ar[i] = (g<<16) + (r<<8) + b
    sm.put(dimmer_ar, 8)
    time.sleep_ms(10)

def pixels_set(i, color):
    ar[i] = (color[1]<<16) + (color[0]<<8) + color[2]

def pixels_fill(color):
    for i in range(len(ar)):
        pixels_set(i, color)

def color_chase(color, wait):
    for i in range(NUM_LEDS):
        pixels_set(i, color)
        time.sleep(wait)
        pixels_show()
    time.sleep(0.2)
 
def wheel(pos):
    # Input a value 0 to 255 to get a color value.
    # The colours are a transition r - g - b - back to r.
    if pos < 0 or pos > 255:
        return (0, 0, 0)
    if pos < 85:
        return (255 - pos * 3, pos * 3, 0)
    if pos < 170:
        pos -= 85
        return (0, 255 - pos * 3, pos * 3)
    pos -= 170
    return (pos * 3, 0, 255 - pos * 3)
 
 
def rainbow_cycle(wait):
    for j in range(255):
        for i in range(NUM_LEDS):
            rc_index = (i * 256 // NUM_LEDS) + j
            pixels_set(i, wheel(rc_index & 255))
        pixels_show()
        time.sleep(wait)

BLACK = (0, 0, 0)
RED = (255, 0, 0)
YELLOW = (255, 150, 0)
GREEN = (0, 255, 0)
CYAN = (0, 255, 255)
BLUE = (0, 0, 255)
PURPLE = (180, 0, 255)
WHITE = (255, 255, 255)
COLORS = (BLACK, RED, YELLOW, GREEN, CYAN, BLUE, PURPLE, WHITE)

print("fills")
for color in COLORS:       
    pixels_fill(color)
    pixels_show()
    time.sleep(0.2)

print("chases")
for color in COLORS:       
    color_chase(color, 0.01)

print("rainbow")
rainbow_cycle(0)
```
pioで正確なタイミングを作っているせいか
今までのやり方のものよりも、安定して光る印象がある。

pio_blink.py
```python

import time
from rp2 import PIO, asm_pio
from machine import Pin

# Define the blink program.  It has one GPIO to bind to on the set instruction, which is an output pin.
# Use lots of delays to make the blinking visible by eye.
@asm_pio(set_init=rp2.PIO.OUT_LOW)
def blink():
    wrap_target()
    set(pins, 1)   [31]
    nop()          [31]
    nop()          [31]
    nop()          [31]
    nop()          [31]
    set(pins, 0)   [31]
    nop()          [31]
    nop()          [31]
    nop()          [31]
    nop()          [31]
    wrap()

# Instantiate a state machine with the blink program, at 1000Hz, with set bound to Pin(25) (LED on the rp2 board)
sm = rp2.StateMachine(0, blink, freq=1000, set_base=Pin(25))

# Run the state machine for 3 seconds.  The LED should blink.
sm.active(1)
#time.sleep(3)
time.sleep(10)
sm.active(0)
```
pio実装によるblinkになる。


pwn_fade.py
```python

# Example using PWM to fade an LED.

import time
from machine import Pin, PWM


# Construct PWM object, with LED on Pin(25).
pwm = PWM(Pin(25))

# Set the PWM frequency.
pwm.freq(1000)

# Fade the LED in and out a few times.
duty = 0
direction = 1
while True:
    for _ in range(8 * 256):
        duty += direction
        if duty > 255:
            duty = 255
            direction = -1
        elif duty < 0:
            duty = 0
            direction = 1
        pwm.duty_u16(duty * duty)
        time.sleep(0.001)
```

## Performance Test for MicroPython

```python

# Peformace Test
import time
def performanceTest():
    msec = time.ticks_ms
    endTime = msec() + 10000
    count = 0
    while msec() < endTime:
        count += 1
    print("Count: ", count)

performanceTest()
```
以下が出力される：  
Count:  1507516

## thonny(pyhton用エディタ)
言うまでもなく、通常のエディタを使ってプログラムすることが可能だが、ここでは専用のエディタをインストールして使用してみる。

以下の手順でインストールする：
```

bash <(curl -s https://thonny.org/installer-for-linux)

# ディスクトップにアイコンができる
```

起動/設定：
```
# 起動
# thonnyアイコンをクリックする

# 設定
(1)実行後、picoボードをPCにUSB接続する
(2)Tools/Options/Interpreterを選択する
 [Which interpreter or device should Thonny use for running your code?]
 のプルダウンメニューから以下を選ぶ：
  「MicroPyton(Raspberry Pi Pico」
(3)「Port」のプルダウンメニューから実際に接続しているポート(/dev/ttyACM0など)を選択する
```

## CircuitPythonダウンロード＆書き込み

以下を手順でファームウェアをダウンロードする：
```

cd ~/Downloads

wget https://downloads.circuitpython.org/bin/raspberry_pi_pico/en_US/adafruit-circuitpython-raspberry_pi_pico-en_US-6.2.0-beta.1.uf2
```
このファームウェアを「ビルドしたファームウェアの書き込み」の手順で書き込む。


## REPL(CircuitPython)
```

Press any key to enter the REPL. Use CTRL-D to reload.

Adafruit CircuitPython 6.2.0-beta.1 on 2021-01-27; Raspberry Pi Pico with rp2040
>>> 
soft reboot

Auto-reload is on. Simply save files over USB to run them or enter REPL to disable.

code.py output:
Hello World!

Code done running.

Press any key to enter the REPL. Use CTRL-D to reload.

Adafruit CircuitPython 6.2.0-beta.1 on 2021-01-27; Raspberry Pi Pico with rp2040
>>> import os
>>> os.uname()
(sysname='rp2040', nodename='rp2040', release='6.2.0', version='6.2.0-beta.1 on 2021-01-27', machine='Raspberry Pi Pico with rp2040')
>>> list(5 * x + y for x in range(10) for y in [4, 2, 1])
[4, 2, 1, 9, 7, 6, 14, 12, 11, 19, 17, 16, 24, 22, 21, 29, 27, 26, 34, 32, 31, 39, 37, 36, 44, 42, 41, 49, 47, 46]
>>> import gc
>>> gc.collect()
>>> gc.mem_free()
220640
>>> 
 
>>> import board
>>> dir(board)
['__class__', 'A0', 'A1', 'A2', 'GP0', 'GP1', 'GP10', 'GP11', 'GP12', 'GP13', 'GP14', 'GP15', 'GP16', 'GP17', 'GP18', 'GP19', 'GP2', 'GP20', 'GP21', 'GP22', 'GP25', 'GP26', 'GP26_A0', 'GP27', 'GP27_A1', 'GP28', 'GP28_A2', 'GP3', 'GP4', 'GP5', 'GP6', 'GP7', 'GP8', 'GP9', 'LED']
```

```
>>> help()
Welcome to Adafruit CircuitPython 6.2.0-beta.1!

Please visit learn.adafruit.com/category/circuitpython for project guides.

To list built-in modules please do `help("modules")`.
>>> help("modules")
__main__          collections       microcontroller   storage
_bleio            digitalio         micropython       struct
_pixelbuf         displayio         msgpack           supervisor
adafruit_bus_device                 errno             neopixel_write    sys
analogio          fontio            os                terminalio
array             framebufferio     pwmio             time
binascii          gamepad           random            touchio
bitbangio         gc                re                ulab
board             io                rp2pio            usb_hid
builtins          json              sdcardio          usb_midi
busio             math              sharpdisplay      vectorio
Plus any modules on the filesystem
>>> 
```

## CircuitPython demo scripts

blink.py
```python

# blink for CircuitPython
from time import sleep
import board
import digitalio

led = digitalio.DigitalInOut(board.LED)
led.direction = digitalio.Direction.OUTPUT

while True:
  led.value = True
  sleep(0.1)
  led.value = False
  sleep(0.1)

```

## Performance Test for CircuitPython

```python

# Peformace Test CircuitPython
from time import monotonic_ns
def performanceTest():
    endTime = monotonic_ns() + 10000000000 # 10 sec
    count = 0
    while monotonic_ns() < endTime:
        count += 1
    print("Count: ", count)

performanceTest()
```
以下が出力される：   
Count:  341033


## 参照情報


・[How to add a reset button to your Raspberry Pi Pico](https://www.raspberrypi.org/blog/how-to-add-a-reset-button-to-your-raspberry-pi-pico/)
```
All you need to do is to wire the GND and RUN pins together and add an extra momentary contact button to your breadboard. Pushing the button will reset the board.
```

・[Programmable I/O with Raspberry Pi Pico](https://www.seeedstudio.com/blog/2021/01/25/programmable-io-with-raspberry-pi-pico/)  


・[Raspberry Pi Picoスタートガイド C/C++](https://pico.raspberrypi.org/files/getting_started.pdf)   
・[Pico C/C++ SDK ](https://pico.raspberrypi.org/files/pico_sdk.pdf)  
・[Pico SDK docs](https://raspberrypi.github.io/pico-sdk-doxygen/index.html)  


・[Pico Python SDK](https://pico.raspberrypi.org/files/pico_python_sdk.pdf)  

・[Pico Pinout](https://www.switch-science.com/catalog/6900/)  


・[Ubuntu 18.04 に Cmake の Latest Release をインストールする](https://qiita.com/comachi/items/d0c1ce5d7b90fe30fced)  

・[NUCLEO-F767ZIにMicropythonをインストールする(v2)](https://beta-notes.way-nifty.com/blog/2020/05/post-940f3b.html)  

・[MicroPython Performance Test](https://beta-notes.way-nifty.com/blog/2020/02/post-5ad431.html)   


以上
