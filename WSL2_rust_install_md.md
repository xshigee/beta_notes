
2021/3/27  

WSL2 Rust Install
# WSL2 Rust Install

## 概要
WSL2でRustを動かす。   
WSL2のUbuntu20.04をホストにする前提で    
PicoボードをターゲットとしてRustをインストールからビルド書き込みまで説明している。  

## Rustのインストール
Ubuntu20.04/WSL2で以下を実行する：

```bash

# 必要なディレクトリを用意する
mkdir ~/rust_ws
mkdir /mnt/c/temp

# Rustをインストールする
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
# 入力を促されるので「1」を入力する
source $HOME/.cargo/env

# 関連ツールのインストール
rustup target add thumbv6m-none-eabi
rustup component add llvm-tools-preview
cargo install uf2conv cargo-binutils
````

## 書き込みスクリプト
WSL2はUSBストレージをサポートしていないので、PowerShell.exe経由で、書き込むスクリプトを用意する。  
pico_UF2.sh  
```bash

#!/bin/bash
uf2conv firmware.bin --base 0x10000000 --family 0xe48bff56 --output firmware.uf2
rm /mnt/c/temp/*
cp firmware.uf2 /mnt/c/temp/
PowerShell.exe copy c:/temp/*.uf2 e:
exit 0
```
・実行属性を与えること。  
・「e:」は環境依存なので、自分の環境で、uf2ファイルを書き込むドライブ・レターを設定すること。  

## Pico用環境をダウンロードする
```

cd ~/rust_ws
git clone https://github.com/jannic/rp-microcontroller-rs.git
```

## ビルド

```bash

cd rp-microcontroller-rs/boards/rp-pico

# binまでビルドする
cargo build --target thumbv6m-none-eabi --example=blink  --release
cargo objcopy --example blink --release -- -O binary firmware.bin
```

## 書き込み
```bash

# BOOTSELボタンを押しながら、PCのUSBコネクタに接続し
# 接続後、BOOTSELボタンを離す。
# RPI-RP2のディレクトリが出現する。
# このタイミングで以下を実行する。
./pico_UF2.sh
```
書き込みが完了するとLEDが点滅する。

## 参考情報
Rust関連：  
[Rustをインストール](https://www.rust-lang.org/ja/tools/install)  
[\`llvm-tools\`, a new rustup component for binary inspection (objdump, nm, size) and profiling (profdata)](https://internals.rust-lang.org/t/llvm-tools-a-new-rustup-component-for-binary-inspection-objdump-nm-size-and-profiling-profdata/7830)  
[Rust support for RP2040 microcontrollers](https://github.com/jannic/rp-microcontroller-rs) 

WSL2関連：  
[WSL2でRDPサーバーを動かす](https://beta-notes.way-nifty.com/blog/2021/03/post-bd6ce0.html)  
[WSL2でSSHサーバーを動かす](https://beta-notes.way-nifty.com/blog/2021/03/post-7e6528.html)  
[WSL2でのplatformioやPico-SDKの利用方法](https://beta-notes.way-nifty.com/blog/2021/02/post-72506a.html)  

以上
