
2020/5/30:  
書き込みツール(bossac)の説明を追加した。  

2020/5/27：  
初版

Rust Circuit-Playground-Express
# Rust Circuit-Playground-Express

## 概要
プログラミング言語Rustを以下のCircuit-Playground-Expressで動かす。
(ホストPCとしてはubuntuを想定している)

[Circuit-Playground-Express](https://www.switch-science.com/catalog/3666/)   

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
cd atsamd/boards/circuit_playground_express

# build
cargo build --example blinky_basic

# conv elf to bin
arm-none-eabi-objcopy -O binary target/thumbv6m-none-eabi/debug/examples/blinky_basic firmware.bin

# conv bin to uf2
uf2conv firmware.bin --base 0x2000 --output firmware.uf2

# flash
# bootloader-modeにする(resetを２度押す)
cp firmware.uf2 /media/USER_NAME/CPLAYBOOT/

```
「USER_NAME」は、自分の環境に合わせて変更すること。
実行結果として、オンボードのLEDが点滅する。

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

(2)bossacによる書き込み
```bash

...
# conv elf to bin
arm-none-eabi-objcopy -O binary target/thumbv6m-none-eabi/debug/examples/blinky_basic firmware.bin
#バイナリを作るところまでは同じ

# flash
# bootloader-modeにする(resetを２度押す)
bossac -p /dev/ttyACM0 -i -e -w -v -R -o 0x2000 firmware.bin

```
出力例：
```bash

$ bossac -p /dev/ttyACM0 -i -e -w -v -R -o 0x2000 firmware.bin
Device       : ATSAMD21x18
Version      : v1.1 [Arduino:XYZ] May 17 2020 17:56:16
Address      : 0x0
Pages        : 4096
Page Size    : 64 bytes
Total Size   : 256KB
Planes       : 1
Lock Regions : 16
Locked       : none
Security     : false
BOD          : true
BOR          : true
Erase flash

Done in 0.756 seconds
Write 13392 bytes to flash (210 pages)
[==============================] 100% (210/210 pages)
Done in 0.085 seconds
Verify 13392 bytes of flash
[==============================] 100% (210/210 pages)
Verify successful
Done in 0.084 seconds
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
