
2020/6/9：  
初版

Rust XIAO
# Rust XIAO

## 概要
プログラミング言語Rustを以下のXIAOで動かす。
(ホストPCとしてはubuntuを想定している)

[Seeeduino XIAO](http://akizukidenshi.com/catalog/g/gM-15178/)   

## 関連ツールのインストール
以下で必要なツールをインスールする：  
(0)bossac(書き込み)ツールのインストール
```bash

sudo apt-get install build-essential
git clone https://github.com/shumatech/BOSSA/tree/arduino
cd BOSSA
make bin/bossac -j4
sudo cp bin/bossac /bin/
```

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

## blinkを実行する
```bash

# download
git clone https://github.com/atsamd-rs/atsamd.git
cd atsamd/boards/xiao_m0

# build
cargo build --example blink

# conv elf to bin
arm-none-eabi-objcopy -O binary target/thumbv6m-none-eabi/debug/examples/blink firmware.bin

# bootloader-modeにする(resetを２度押す)

# flash
bossac -p /dev/ttyACM0 -i -e -w -v -R -o 0x2000 firmware.bin

```
実行結果として、オンボードの３つのLEDが点滅する。
(グリーンのLEDは通電表示のLEDなので点滅せずに光っている)

出力例：
```bash

$ bossac -p /dev/ttyACM0 -i -e -w -v -R -o 0x2000 firmware.bin

Device       : ATSAMD21x18
Version      : v1.1 [Arduino:XYZ] Nov 27 2019 16:35:59
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

Done in 0.834 seconds
Write 13260 bytes to flash (208 pages)
[==============================] 100% (208/208 pages)
Done in 0.087 seconds
Verify 13260 bytes of flash
[==============================] 100% (208/208 pages)
Verify successful
Done in 0.080 seconds
```

## usb_serial
```bash

# download
git clone https://github.com/atsamd-rs/atsamd.git
cd atsamd/boards/xiao_m0

# build
cargo build --example usb_serial --features="usb"

# conv elf to bin
arm-none-eabi-objcopy -O binary target/thumbv6m-none-eabi/debug/examples/usb_serial firmware.bin

# bootloader-modeにする(resetを２度押す)

# flash
bossac -p /dev/ttyACM0 -i -e -w -v -R -o 0x2000 firmware.bin

```
書き込みが終了すると、USBシリアルのエコープログラムが動作する。   
「picocom /dev/ttyACM0 -b115200」で通信プログラムを起動して
キー入力すると、入力したキーが表示される。キーの入力に応じて、
オンボードの青いLEDが点滅する。


## 参照情報

[atsamd & atsame support for Rust](https://github.com/atsamd-rs/atsamd)  
[Running bossac on the command line](https://learn.adafruit.com/adafruit-feather-m0-express-designed-for-circuit-python-circuitpython/uf2-bootloader-details)   

[STM32 Nucleo Board STM32F103RB](http://akizukidenshi.com/catalog/g/gM-07724/)   

[The Embedded Rust Book](https://rust-embedded.github.io/book/intro/index.html)  
[The Embedded Rust Book - Semihosting](https://rust-embedded.github.io/book/start/semihosting.html)  

[UbuntuにRustをインストールする](https://qiita.com/yoshiyasu1111/items/0c3d08658560d4b91431)   

[Adding a new board](https://github.com/atsamd-rs/atsamd/wiki/Adding-a-new-board)  


以上
