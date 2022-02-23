
2021/2/24  
ソース(morse_blinky.c)修正。

2021/2/6  

rpi Pico Project Generator
# rpi Pico Project Generator

## 概要
以下のPicoボードにProject_Generatorを使ってC言語をビルドする。  
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
任意のプログラムを入力する(例：morse_blinky)

(2)Location:
プロジェクトを置くディレクトリを指定する(例：~/pico_ws)

(3)Feature:
利用する機能にチェックを入れる

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

```

cd ~/pico_ws
# 自動生成したプロジェクトを開く
cd morse_blinky
gedit CMakeList.txt
# 内容に以下が含まれていることを確認する
# 存在していない場合、追加する
```
```
# enable usb output, disable uart output
pico_enable_stdio_usb(morse_blinky 1)
pico_enable_stdio_uart(morse_blinky 0)
```
「morse_blinky」のところは、設定したプロジェクト名になる。

## プログラムを編集する

```

cd ~/pico_ws
# 自動生成したプロジェクトを開く
cd morse_blinky
gedit morse_blinky.c
```

以下の内容になるように編集する：
```

// modified on 2021/2/6
/**
 * Copyright (c) 2020 Raspberry Pi (Trading) Ltd.
 *
 * SPDX-License-Identifier: BSD-3-Clause
 */

#include <stdio.h>
#include "pico/stdlib.h"
#include "hardware/gpio.h"

char str[100];

const uint LED_PIN = 25;
const uint DOT_PERIOD_MS = 100;

const char *morse_letters[] = {
        ".-",    // A
        "-...",  // B
        "-.-.",  // C
        "-..",   // D
        ".",     // E
        "..-.",  // F
        "--.",   // G
        "....",  // H
        "..",    // I
        ".---",  // J
        "-.-",   // K
        ".-..",  // L
        "--",    // M
        "-.",    // N
        "---",   // O
        ".--.",  // P
        "--.-",  // Q
        ".-.",   // R
        "...",   // S
        "-",     // T
        "..-",   // U
        "...-",  // V
        ".--",   // W
        "-..-",  // X
        "-.--",  // Y
        "--.."   // Z
};

void put_morse_letter(const char *pattern) {
    for (; *pattern; ++pattern) {
        gpio_put(LED_PIN, 1);
        if (*pattern == '.')
            sleep_ms(DOT_PERIOD_MS);
        else
            sleep_ms(DOT_PERIOD_MS * 3);
        gpio_put(LED_PIN, 0);
        sleep_ms(DOT_PERIOD_MS * 1);
    }
    sleep_ms(DOT_PERIOD_MS * 2);
}

void put_morse_str(const char *str) {
    for (; *str; ++str) {
        //if (*str >= 'A' && *str < 'Z') {
        if (*str >= 'A' && *str <= 'Z') {
            put_morse_letter(morse_letters[*str - 'A']);
        //} else if (*str >= 'a' && *str < 'z') {
        } else if (*str >= 'a' && *str <= 'z') {
            put_morse_letter(morse_letters[*str - 'a']);
        } else if (*str == ' ') {
            sleep_ms(DOT_PERIOD_MS * 4);
        }
    }
}

int main() {
    stdio_init_all();

    gpio_init(LED_PIN);
    gpio_set_dir(LED_PIN, GPIO_OUT);
    while (true) {
        printf("Enter String for Morse\n");
        fflush(stdin);
        scanf("%s",str);
        //gets(str); // does not work??

        printf("morse: %s\n", str);
        //puts(str);

        //put_morse_str("Hello world");
        put_morse_str(str);
        //sleep_ms(1000);
    }
}
```
これは提供されているサンプルを変更したものになる。

## ビルド/書き込み
以下の手順でビルド/書き込みできる：

```

cd ~/pico_ws/morse_blinky
cd build
make -j4

# BOOTSELボタンを押しながらPicoボードをホストPCにUSB接続する。
# BOOTSELボタンを離す。
# 書き込み用のRPI-RP2ディレクトリが現れる

cp *.uf2 /media/<USER>/RPI-RP2/
```
以上でファームウェア(.uf2)がPicoボードに書き込まれる。  
(\<USER\>の部分はホストの環境に合わせる)

なお、ソースを修正する場合、以下のようにする：
```

gedit ../morse_blinky.c 
```

## 動作確認
picocom(またはbt)を使いPicoボードとPC間でUSBシリアルで通信する。  

通信例：
```

$ picocom  -b115200 /dev/ttyACM0
<省略>

# 最初に適当な文字を入力する
morse: adf
# 以降、入力した文字列のモールス信号がボード上のLEDで点滅する
Enter String for Morse
morse: kool
Enter String for Morse
morse: pool
Enter String for Morse
```

## 参照情報

・[How to add a reset button to your Raspberry Pi Pico](https://www.raspberrypi.org/blog/how-to-add-a-reset-button-to-your-raspberry-pi-pico/)  
All you need to do is to wire the GND and RUN pins together and add an extra momentary contact button to your breadboard. Pushing the button will reset the board.


・[Programmable I/O with Raspberry Pi Pico](https://www.seeedstudio.com/blog/2021/01/25/programmable-io-with-raspberry-pi-pico/)  


・[Raspberry Pi Picoスタートガイド C/C++](https://pico.raspberrypi.org/files/getting_started.pdf)   
・[Pico C/C++ SDK ](https://pico.raspberrypi.org/files/pico_sdk.pdf)  
・[Pico SDK docs](https://raspberrypi.github.io/pico-sdk-doxygen/index.html)  


・[Raspberry Pi Pico Datasheet](https://pico.raspberrypi.org/files/pico_datasheet.pdf)  
・[RP2040 Datasheet](https://pico.raspberrypi.org/files/rp2040_datasheet.pdf)  


・[Raspberry Pi Pico Pinout](https://akizukidenshi.com/download/ds/raspberry/Pico-R3-A4-Pinout.pdf)  


・[Ubuntu 18.04 に Cmake の Latest Release をインストールする](https://qiita.com/comachi/items/d0c1ce5d7b90fe30fced)  

以上
