2020/5/30:  
書き込みツール(bossac)の説明を追加した。  

2020/5/28  
初版  

Rust Feather-M4-Express
# Rust Feather-M4-Express

## 概要
プログラミング言語Rustを以下のFeather-M4-Expressで動かす。
(ホストPCとしてはubuntuを想定している)

[Feather-M4-Express](https://www.switch-science.com/catalog/5887/)   

## 関連ツールのインストール
以下で必要なツールをインスールする：  
(1)rustツールのインストール
```bash

curl https://sh.rustup.rs -sSf | sh
```

(2)Cコンパイラのバージョンアップ   
binを作成するのに、これに含まれるツールを使用するのでインストールする
```bash
以下でインストールしたコンパイラが古くてビルドエラーが出たのでバージョンアップする。
sudo apt-get install gcc-arm-none-eabi

(1)以下のurlから最新版をダウンロードする：
https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads

解凍したものを以下のフォルダに置く：
$HOME/Downloads/gcc-arm-none-eabi-8-2019-q3-update

(2)パス設定
#古いコンパイラを削除する
sudo apt-get remove gcc-arm-none-eabi
#パスを設定する
export PATH=$PATH:$HOME/Downloads/gcc-arm-none-eabi-8-2019-q3-update/bin

```
exportは、.bashrcに登録する

(3)picocom(通信ソフト)のインストール
```bash


sudo apt-get install picocom

```

(4)rust関連ツールのインストール
```bash

# for samd11, samd21:
rustup target add thumbv6m-none-eabi
# for samd51, same54:
rustup target add thumbv7em-none-eabihf
# uf2conv etc install
rustup component add llvm-tools-preview
cargo install uf2conv cargo-binutils
```

## blinkyを実行する
```bash

# download
git clone https://github.com/atsamd-rs/atsamd.git
cd atsamd/boards/feather_m4/feather_m4

# build
cargo build --example blinky_basic

# conv elf to bin
arm-none-eabi-objcopy -O binary target/thumbv7em-none-eabihf/debug/examples/blinky_basic firmware.bin

# conv bin to uf2
uf2conv firmware.bin --base 0x4000 --output firmware.uf2

# flash
# bootloader-modeにする(resetを２度押す)
cp firmware.uf2 /media/USER_NAME/FEATHERBOOT/

```
「USER_NAME」は、自分の環境に合わせて変更すること。  
実行結果として、オンボードのLEDが点滅する。

## オンボードのnexpixelを動かす
```bash

# build
cargo build --example neopixel_rainbow
cargo build --example neopixel_rainbow　--release
# debugでビルドした後、releaseでビルドする必要がある(そうしないととエラーになる)

# conv elf to bin
arm-none-eabi-objcopy -O binary target/thumbv7em-none-eabihf/release/examples/neopixel_rainbow firmware.bin
# 性能問題があるのでdebugではなくreleaseを採用する

# conv bin to uf2
uf2conv firmware.bin --base 0x4000 --output firmware.uf2

# flash
# bootloader-modeにする(resetを２度押す)
cp firmware.uf2 /media/USER_NAME/FEATHERBOOT/

```
「USER_NAME」は、自分の環境に合わせて変更すること。  
実行結果として、オンボードのneopixelが色を変えながら光る。

## serialを動かす
```bash

# build
cargo build --example serial

# conv elf to bin
arm-none-eabi-objcopy -O binary target/thumbv7em-none-eabihf/debug/examples/serial firmware.bin

# conv bin to uf2
uf2conv firmware.bin --base 0x4000 --output firmware.uf2

# flash
# bootloader-modeにする(resetを２度押す)
cp firmware.uf2 /media/USER_NAME/FEATHERBOOT/

```
「USER_NAME」は、自分の環境に合わせて変更すること。  
USBシリアルを、RX,TX,GNDと接続して、実行すると
文字列がシリアル出力される。

出力例：
```bash

picocom /dev/ttyUSB0 -b19200

...
Terminal ready
Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hello, world!Hell
...

```
ソース上の文字列に改行などが入っていないので、'Hello, world!'が繋がって出力される。

注意：シリアルが動作中だと「bootloader-mode」に入りにくいようだ。
その場合、resetを押した後、再度、resetを２度押すと、そのモードに入るようだ。

## serial_echoを動かす
```bash

# build
cargo build --example uart_poll_echo

# conv elf to bin
arm-none-eabi-objcopy -O binary target/thumbv7em-none-eabihf/debug/examples/uart_poll_echo firmware.bin

# conv bin to uf2
uf2conv firmware.bin --base 0x4000 --output firmware.uf2

# flash
# bootloader-modeにする(resetを２度押す)
cp firmware.uf2 /media/USER_NAME/FEATHERBOOT/

```
「USER_NAME」は、自分の環境に合わせて変更すること。  
USBシリアルを、RX,TX,GNDと接続して、実行して
通信ソフト「picocom /dev/ttyUSB0 -b19200」を起動する。
そこで、文字をタイプすると、そのまま、文字が表示される。.　  

注意：シリアルが動作中だと「bootloader-mode」に入りにくいようだ。
その場合、resetを押した後、再度、resetを２度押すと、そのモードに入るようだ。

## timerを動かす
```bash

# build
cargo build --example timers

# conv elf to bin
arm-none-eabi-objcopy -O binary target/thumbv7em-none-eabihf/debug/examples/timers firmware.bin

# conv bin to uf2
uf2conv firmware.bin --base 0x4000 --output firmware.uf2

# flash
# bootloader-modeにする(resetを２度押す)
cp firmware.uf2 /media/USER_NAME/FEATHERBOOT/

```
「USER_NAME」は、自分の環境に合わせて変更すること。   
実行結果として、タイマーの周期でオンボードのLEDが点滅する。  
なお、周期がわかりにくかったので、ソースの「timer.start(50u32.hz());」を
「timer.start(5u32.hz());」に変更した。


## 乱数生成を動かす
```bash

# build
cargo build --example trng

# conv elf to bin
arm-none-eabi-objcopy -O binary target/thumbv7em-none-eabihf/debug/examples/trng firmware.bin

# conv bin to uf2
uf2conv firmware.bin --base 0x4000 --output firmware.uf2

# flash
# bootloader-modeにする(resetを２度押す)
cp firmware.uf2 /media/USER_NAME/FEATHERBOOT/

```
「USER_NAME」は、自分の環境に合わせて変更すること。  
実行結果として、生成した乱数の周期でオンボードのLEDが点滅する。

## bossacによる書き込み
uf2ファイルに変換せずにbinのまま書き込みができるbossacというツールがあるので
それの使い方について説明する。   
(1)bossacツールのインストール
```bash

sudo apt-get install build-essential
git clone https://github.com/shumatech/BOSSA/tree/arduino
cd BOSSA
make bin/bossac -j4
sudo cp bin/bossac /bin/
```

(2)bossacによる書き込み(例)
```bash


# build
cargo build --example blinky_basic

# conv elf to bin
arm-none-eabi-objcopy -O binary target/thumbv7em-none-eabihf/debug/examples/blinky_basic firmware.bin
#バイナリを作るところまでは同じ

# flash
# bootloader-modeにする(resetを２度押す)
bossac -p /dev/ttyACM0 -i -e -w -v -R -o 0x4000 firmware.bin

```
出力例：
```bash

$ bossac -p /dev/ttyACM0 -i -e -w -v -R -o 0x4000 firmware.bin
Device       : ATSAMD51x19
Version      : v1.1 [Arduino:XYZ] May 17 2020 17:56:23
Address      : 0x0
Pages        : 1024
Page Size    : 512 bytes
Total Size   : 512KB
Planes       : 1
Lock Regions : 32
Locked       : none
Security     : false
BOD          : false
BOR          : true
Erase flash

Done in 2.383 seconds
Write 13852 bytes to flash (28 pages)
[==============================] 100% (28/28 pages)
Done in 0.072 seconds
Verify 13852 bytes of flash
[==============================] 100% (28/28 pages)
Verify successful
Done in 0.029 seconds
```

## 参照情報

[atsamd & atsame support for Rust](https://github.com/atsamd-rs/atsamd)  
[Running bossac on the command line](https://learn.adafruit.com/adafruit-feather-m0-express-designed-for-circuit-python-circuitpython/uf2-bootloader-details)   

[STM32 Nucleo Board STM32F103RB](http://akizukidenshi.com/catalog/g/gM-07724/)   

[The Embedded Rust Book](https://rust-embedded.github.io/book/intro/index.html)  
[The Embedded Rust Book - Semihosting](https://rust-embedded.github.io/book/start/semihosting.html)  

[UbuntuにRustをインストールする](https://qiita.com/yoshiyasu1111/items/0c3d08658560d4b91431)   

[Adding a new board](https://github.com/atsamd-rs/atsamd/wiki/Adding-a-new-board)  

以上

