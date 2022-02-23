
2021/2/7  

rpi Pico Project Generator for PIO
# rpi Pico Project Generator for PIO

## 概要
以下のPicoボードにProject_Generatorを使ってpioを含むC言語をビルドする。  
「[PicoボードにPico_SDKでC言語をビルドする](https://beta-notes.way-nifty.com/blog/2021/02/post-2fff25.html)」でのインストールが既に完了している前提でホストPCとしてはubuntuを想定している。

・[Raspberry Pi Pico](https://www.switch-science.com/catalog/6900/)  


## ターミナルソフト(bt)のインストール
別のターミナルソフトがインストール済みであれば特にインストールする必要はない。

以下の手順でインストールする：
```
mkdir bt
cd bt
git clone https://github.com/wtarreau/bootterm
cd bootterm
make
sudo make install
```
使い方(例)
```
# 以下のコマンドで接続しているシリアルのデバイス名が表示される

$ bt -l
 port |  age (sec) | device     | driver           | description          
------+------------+------------+------------------+----------------------
    0 |       2373 | ttyS4      | serial           |                  
 *  1 |        332 | ttyACM0    | cdc_acm          | CircuitPython CDC control 

# 以下のコマンドで、最近、有効化されたシリアルデバイスに接続する
$ bt
No port specified, using ttyACM0 (last registered). Use -l to list ports.
Trying port ttyACM0... Connected to ttyACM0 at 115200 bps.
Escape character is 'Ctrl-]'. Use escape followed by '?' for help.
```

参照：  
[Bootterm – a developer-friendly serial terminal program](https://www.cnx-software.com/2020/12/14/bootterm-a-developer-friendly-serial-terminal-program/)  



## プロジェクトを自動生成する
プログラムの雛形を自動生成できるので使ってみる。   
以下の手順でgeneratorを動かす：
```

cd pico
# 以下の1行は、言うまでもなく、一回実行すれば良い
git clone https://github.com/raspberrypi/pico-project-generator.git

export PICO_SDK_PATH=~/pico/pico-sdk
cd pico-project-generator
./pico_project.py --gui
```
以上でgeneratorのGUIが起動する。

以下のようにパラメータを入力する：
```

(2)Project Name:
任意のプログラムを入力する(例：pio_blink)

(2)Location:
プロジェクトを置くディレクトリを指定する(例：~/pico_ws)

(3)Feature:
pioを使用するので[PIO interface]をチェックする。

(4)Console Option:
コンソールをUSBシリアルにするので、以下を選択する：
[Console over USB]

(5)Code Option:
そのままで良い。

(6)Build Option:
そのままで良い。

(7)パラメータを入力したら
[OK]をクリックするとプロジェクトが生成される。

(3)generatorから抜けるとき
[Quit]をクリックする
```

## CMakeList.txtを確認する
generatorにバグがあるようなので意図どおりに設定できているか確認する：
```

cd ~/pico_ws
# 自動生成したプロジェクトを開く
cd pio_blink
gedit CMakeList.txt
# 内容に以下が含まれていることを確認する
# 存在していない場合は、追加する
# 0,1が逆になっている場合は、修正する
```
```
pico_enable_stdio_usb(pio_blink 1)
pico_enable_stdio_uart(pio_blink 0)
```
「pio_blink」のところは、設定したプロジェクト名になる。

## プログラムを編集する

```

cd ~/pico_ws
# 自動生成したプロジェクトを開く
cd pio_blink
gedit pio_blink.c
```

以下の内容になるように編集する：
```
// modified on 2021/2/7
/**
 * Copyright (c) 2020 Raspberry Pi (Trading) Ltd.
 *
 * SPDX-License-Identifier: BSD-3-Clause
 */

#include <stdio.h>

#include "pico/stdlib.h"
#include "hardware/pio.h"
#include "pio_blink.pio.h"

void blink_pin_forever(PIO pio, uint sm, uint offset, uint pin, uint freq);

char str[10];

int main() {
    //setup_default_uart();
    stdio_init_all();

    // wait for string typed
    printf("Type any string to start.\n");
    fflush(stdin);
    scanf("%s",str);

    // todo get free sm
    PIO pio = pio0;
    uint offset = pio_add_program(pio, &pio_blink_program);
    printf("Loaded program at %d\n", offset);

    //blink_pin_forever(pio, 0, offset, 0, 3);
    //blink_pin_forever(pio, 0, offset, 25, 3); // on-board LED
    //blink_pin_forever(pio, 0, offset, 25, 4); // on-board LED
    blink_pin_forever(pio, 0, offset, 25, 1); // on-board LED
    blink_pin_forever(pio, 1, offset, 6, 4);
    blink_pin_forever(pio, 2, offset, 11, 1);
}

void blink_pin_forever(PIO pio, uint sm, uint offset, uint pin, uint freq) {
    blink_program_init(pio, sm, offset, pin);
    pio_sm_set_enabled(pio, sm, true);

    printf("Blinking pin %d at freq %d\n", pin, freq);
    pio->txf[sm] = 24000000 / freq;
}
```
これは提供されているサンプルを修正したものになる。

以下の部分は実際のプログラム名(pio_blink)に合わせる：
```

#include "pio_blink.pio.h"

pio_add_program(pio, &pio_blink_program);
```

## pioプログラムの追加
以下のファイルを同じディレクトリに追加する：
pio_blink.pio
```

;
; Copyright (c) 2020 Raspberry Pi (Trading) Ltd.
;
; SPDX-License-Identifier: BSD-3-Clause
;

; SET pin 0 should be mapped to your LED GPIO

.program pio_blink
    pull block
    out y, 32
.wrap_target
    mov x, y
    set pins, 1   ; Turn LED on
lp1:
    jmp x-- lp1   ; Delay for (x + 1) cycles, x is a 32 bit number
    mov x, y
    set pins, 0   ; Turn LED off
lp2:
    jmp x-- lp2   ; Delay for the same number of cycles again
.wrap             ; Blink forever!


% c-sdk {
// this is a raw helper function for use by the user which sets up the GPIO output, and configures the SM to output on a particular pin

void blink_program_init(PIO pio, uint sm, uint offset, uint pin) {
   pio_gpio_init(pio, pin);
   pio_sm_set_consecutive_pindirs(pio, sm, pin, 1, true);
   pio_sm_config c = pio_blink_program_get_default_config(offset);
   sm_config_set_set_pins(&c, pin, 1);
   pio_sm_init(pio, sm, offset, &c);
}
%}
```
以下の部分は実際のプログラム名(pio_blink)にあわせる：
```

.program pio_blink

pio_blink_program_get_default_config
```

## pioヘッダーを作成する
以下の手順で \*.pioから \*.pio.hを作成する：
```

pioasm pio_blink.pio > pio_blink.pio.h
```

## ビルド/書き込み
ここまで完了すると後は、  
以下の手順でビルド/書き込みできる：

```

cd ~/pico_ws/pio_blink
cd build
make -j4

# BOOTSELボタンを押しながらPicoボードをホストPCにUSB接続する。
# BOOTSELボタンを離す。
# 書き込み用のRPI-RP2ディレクトリが現れる

cp *.uf2 /media/<USER>/RPI-RP2/
```
以上でファームウェア(.uf2)がPicoボードに書き込まれる。  
(\<USER\>の部分はホストの環境に合わせる)

## 動作確認
picocom(またはbt)を使いPicoボードとPC間でUSBシリアルで通信する。  

通信例：
```

$ picocom  -b115200 /dev/ttyACM0
<省略>

# 最初に適当な文字列を入力する
Loaded program at 24
Blinking pin 25 at freq 1
Blinking pin 6 at freq 4
Blinking pin 11 at freq 1
```
pin25はボード上のLEDなのでLEDが点滅する。
pin6,11は、ブレッドボードなどでLEDを接続してあれば
別の周期で点滅する。

## 参照情報

・[Programmable I/O with Raspberry Pi Pico](https://www.seeedstudio.com/blog/2021/01/25/programmable-io-with-raspberry-pi-pico/)  


・[How to add a reset button to your Raspberry Pi Pico](https://www.raspberrypi.org/blog/how-to-add-a-reset-button-to-your-raspberry-pi-pico/)
```
All you need to do is to wire the GND and RUN pins 
together and add an extra momentary contact button 
to your breadboard. 
Pushing the button will reset the board.
```

・[Programmable I/O with Raspberry Pi Pico](https://www.seeedstudio.com/blog/2021/01/25/programmable-io-with-raspberry-pi-pico/)  


・[Raspberry Pi Picoスタートガイド C/C++](https://pico.raspberrypi.org/files/getting_started.pdf)   
・[Pico C/C++ SDK ](https://pico.raspberrypi.org/files/pico_sdk.pdf)  
・[Pico SDK docs](https://raspberrypi.github.io/pico-sdk-doxygen/index.html)  


・[Raspberry Pi Pico Datasheet](https://pico.raspberrypi.org/files/pico_datasheet.pdf)  
・[RP2040 Datasheet](https://pico.raspberrypi.org/files/rp2040_datasheet.pdf)  


・[Raspberry Pi Pico Pinout](https://akizukidenshi.com/download/ds/raspberry/Pico-R3-A4-Pinout.pdf)  


・[Ubuntu 18.04 に Cmake の Latest Release をインストールする](https://qiita.com/comachi/items/d0c1ce5d7b90fe30fced)  

以上
