
2020/5/9

Rust Longan Nano
# Rust Longan Nano

## 概要
プログラミング言語Rustを以下のLongan-nanoボードで動かす。
(ホストPCとしてはubuntuを想定している)
  
[Sipeed Longan Nano RISC-V GD32VF103CBT6開発ボード](http://akizukidenshi.com/catalog/g/gK-14678/)   

## 関連ツールのインストール
以下で必要なツールをインスールする：  
(1)rustツールのインストール
```bash

curl https://sh.rustup.rs -sSf | sh
```

(2)その他の必要ツールのインストール
```bash

sudo apt-get install picocom

```

## Rust(RISC-V)関係ツールのインストール

(1)Installing dependencies

```bash

rustup default stable
rustup target add riscv32imac-unknown-none-elf
rustup default nightly
rustup target add riscv32imac-unknown-none-elf
```

(2)RISC-V toolchain (e.g. from SiFive) install
```bash

mkdir riscv-gnu-toolchain
cd riscv-gnu-toolchain
wget https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz
tar -zxvf riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz
export PATH=$PATH:~/riscv-gnu-toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin

```

(3)GD32対応(GD32VF103 support)のdfu-utilのインストール
```bash

mkdir gd32-tools
cd gd32-tools
git clone https://github.com/riscv-mcu/gd32-dfu-utils.git
cd gd32-dfu-utils
./autogen.sh
./configure
make
sudo cp src/dfu-util /opt/bin/gd32-dfu-util

export PATH=$PATH:/opt/bin
```
「dfu-util 0.9」でGD32対応済みの可能性があるが
ここでは、GD32でフォークしたものを使用する。

(参考)古いバージョンでの問題点:    
https://github.com/riscv-rust/longan-nano/issues/5


以上で、exportとしたものは、.bashrcに登録すること。


## Build
以下を実行してサンプルコードをビルドする。
```bash

git clone https://github.com/riscv-rust/longan-nano.git
cd longa-nano

nano .cargo/config
以下のように修正する：
"link-arg=-Tmemory-c8.x" →　"link-arg=-Tmemory-cb.x"

# To build all the provided examples
cargo build --examples --release --all-features

```

## make bin & flash it
以下を実行してbin(複数)を作成してボードに書き込む。
```bash

riscv64-unknown-elf-objcopy -O binary target/riscv32imac-unknown-none-elf/release/examples/blinky blinky.bin
riscv64-unknown-elf-objcopy -O binary target/riscv32imac-unknown-none-elf/release/examples/ferris ferris.bin
riscv64-unknown-elf-objcopy -O binary target/riscv32imac-unknown-none-elf/release/examples/display display.bin
riscv64-unknown-elf-objcopy -O binary target/riscv32imac-unknown-none-elf/release/examples/hello_world hello_world.bin
riscv64-unknown-elf-objcopy -O binary target/riscv32imac-unknown-none-elf/release/examples/scan scan.bin

# Keep the BOOT0 button pressed while power-up or 
# while pressing and releaseing the reset button. 
#以下から実行したいbinを１つ選び実行する：
gd32-dfu-util -a 0 -s 0x08000000:leave -D blinky.bin
gd32-dfu-util -a 0 -s 0x08000000:leave -D ferris.bin
gd32-dfu-util -a 0 -s 0x08000000:leave -D display.bin
gd32-dfu-util -a 0 -s 0x08000000:leave -D hello_world.bin
gd32-dfu-util -a 0 -s 0x08000000:leave -D scan.bin

```
書き込み後、(再起動しないようなので)[reset]ボタンを押すこと。


## 実行結果
(1)blinky  
実行すると、LEDが赤、緑、青と繰り返し変化して光る。  

(2)ferris  
実行すると、LCDに、ferris(蟹)の画像が表示される。  
蟹がRust言語のマスコットになっているらしい。。。

以下にＦｅｒｒｉｓの画像がある：  
https://rustacean.net/  

(3)display  
実行すると、LCDに" Hello Rust! "のテキストが表示される。  

(4)hello_world  
USBシリアルを接続して、実行すると以下のように"Hello, world"がシリアルに出力される。 
```

picocom /dev/ttyUSB0 -b115200
Terminal ready

Hello, world
```

(5)scan  
USBシリアルを接続して、実行すると以下がシリアルに出力される。
```

picocom /dev/ttyUSB0 -b115200
Terminal ready

scan started
00000000..20007fff
40000000..40017fff
40020000..400207ff
40021000..400213ff
40022000..400223ff
40023000..400233ff
50000000..5003ffff
60000000..a0000fff
d1000000..d1000fff
d2000000..d200ffff
e0000000..efffffff
==> ffffff00
scan finished
```

なお、USBシリアルは、GNG/T0/R0にクロスで接続する。(3V3は接続しない)  


## 参照情報

[Sipeed Longan Nano RISC-V GD32VF103CBT6開発ボード](http://akizukidenshi.com/catalog/g/gK-14678/)   

[The Embedded Rust Book](https://rust-embedded.github.io/book/intro/index.html)  
[The Embedded Rust Book - Semihosting](https://rust-embedded.github.io/book/start/semihosting.html)  

[UbuntuにRustをインストールする](https://qiita.com/yoshiyasu1111/items/0c3d08658560d4b91431)   

以上
