
2020/5/8

Rust STM32F4xx
# Rust STM32F4xx

## 概要
プログラミング言語Rustを以下のNucleo-F401REボードで動かす。
(ホストPCとしてはubuntuを想定している)
  
[STM32 Nucleo Board STM32F401](http://akizukidenshi.com/catalog/g/gM-07723/)   

## 関連ツールのインストール
以下で必要なツールをインスールする：  
(0)rustツールのインストール
```bash

curl https://sh.rustup.rs -sSf | sh
```
(1)書き込みツール(st-flash)のインストール
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

(3)その他の必要ツールのインストール
```bash

sudo apt-get install openocd
sudo apt-get install picocom

```

## sample#0( gpio_hal_blinky)をビルドして実行する
最初の設定がターゲットボードとしてSTM32F401REになっているので、そのまま動かす。

```bash

git clone https://github.com/jkristell/nucleo-f401re.git
cd nucleo-f401re
rustup target add thumbv7em-none-eabihf

cargo clean
cargo build --example gpio_hal_blinky

arm-none-eabi-objcopy -O binary target/thumbv7em-none-eabihf/debug/examples/gpio_hal_blinky firmware.bin
#ボードのUSBコネクタをホストPCに接続する
st-flash write firmware.bin 0x8000000

```
正常に動作した場合、ボードのグリーンLEDが点滅する。

## sample#1(delay-blinky)をビルドして実行する
上とは別のgitから持ってきたものを実行してみる。  
ターゲットボードをSTM32F401REにするために
memory.xを以下に変更する。   
(gitでダウンロード後)

memory.x
```

MEMORY
{
  /* NOTE K = KiBi = 1024 bytes */
  FLASH : ORIGIN = 0x08000000, LENGTH = 512K
  RAM : ORIGIN = 0x20000000, LENGTH = 96K
}

/* This is where the call stack will be allocated. */
/* The stack is of the full descending type. */
/* NOTE Do NOT modify `_stack_start` unless you know what you are doing */
_stack_start = ORIGIN(RAM) + LENGTH(RAM);

```

以下の手順を実行する。  
(memory.xは、上の内容に変更すること)
```bash

git clone https://github.com/stm32-rs/stm32f4xx-hal.git
cd stm32f4xx-hal

rustup target add thumbv7em-none-eabihf

cargo clean
cargo build --example delay-blinky --release --features="rt stm32f411"

# elfをbinに変換する
arm-none-eabi-objcopy -O binary target/thumbv7em-none-eabihf/release/examples/delay-blinky firmare.bin

# binを書き込む
#ボードのUSBコネクタをホストPCに接続する
st-flash write firmware.bin 0x8000000

以下のようなエラーがでて書き込めない場合、OpenOCDで書き込む。
$ st-flash write firmware.bin 0x8000000
st-flash 1.6.0
2020-05-08T20:53:06 INFO common.c: Loading device parameters....
2020-05-08T20:53:06 INFO common.c: Device connected is: F4 device (Dynamic Efficency), id 0x10006433
2020-05-08T20:53:06 INFO common.c: SRAM size: 0x18000 bytes (96 KiB), Flash: 0x80000 bytes (512 KiB) in pages of 16384 bytes
open(firmware.bin) == -1
2020-05-08T20:53:06 ERROR common.c: map_file() == -1
stlink_fwrite_flash() == -1

```
正常に動作した場合、ボードのグリーンLEDが点滅する。(sample#0と同様)

## OpenOCDによる書き込み
今までは、st-flashで書き込んだが、以下の手順でOpenOCDで書き込むこともできる。   
別の端末で以下を実行する：
```bash

openocd -f interface/stlink-v2-1.cfg -f target/stm32f4x.cfg
```
エラーになった場合、stlinkのファームウェアが古い可能性があるので
そのときは「tlink-v2-1.cfg」を「tlink-v2.cfg」に変更して実行する。
実行するとopenocdがサーバーとして動作して、gdb(やtelnet)を待ち受ける状態になる。

buildの作業で使っている端末で以下を実行する：
(delay-blinkyの場合)
```bash

cargo run --example delay-blinky --release --features="rt stm32f411"
#ビルドが完了すると自動的にgdbが起動するので
#以下の手順でopenocdと接続してファームウェアを書き込む。
(gdb) target remote :3333
Remote debugging using :3333
0x00000000 in ?? ()
(gdb) load
Loading section .vector_table, size 0x198 lma 0x8000000
Loading section .text, size 0x50c lma 0x8000198
Loading section .rodata, size 0x13c lma 0x80006b0
Start address 0x800055c, load size 2016
Transfer rate: 3 KB/sec, 672 bytes/write.
(gdb) quit
Quit anyway? (y or n) y
#[reset]ボタンは無効になっているようなので、
#USB接続を切って電源をオフした後、再度、USBを接続して再起動すること

```
openocdの起動用スクリプトを設定すると書き込みを自動化できるようだが、ここでは、それを使用していない。


## ターゲット情報

Cortex-Mプロセッサとターゲットは以下の対応になっている：
```

thumbv6m-none-eabi - Cortex-M0 と M0+
thumbv7m-none-eabi - Cortex M3
thumbv7em-none-eabi - Cortex M4 と M7、FPU なし
thumbv7em-none-eabihf - Cortex M4 と M7、FPU あり

```

## 参照情報

[STM32 Nucleo Board STM32F401](http://akizukidenshi.com/catalog/g/gM-07723/)   

[The Embedded Rust Book](https://rust-embedded.github.io/book/intro/index.html)  
[The Embedded Rust Book - Semihosting](https://rust-embedded.github.io/book/start/semihosting.html)  

[UbuntuにRustをインストールする](https://qiita.com/yoshiyasu1111/items/0c3d08658560d4b91431)   

以上
