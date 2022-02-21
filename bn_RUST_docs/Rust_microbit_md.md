
2020/5/10

Rust micro:bit
# Rust micro:bit

## 概要
プログラミング言語Rustを以下のmicro:bitボードで動かす。
(ホストPCとしてはubuntuを想定している)

[micro:bit](https://www.switch-science.com/catalog/5263/)  

## 関連ツールのインストール
以下で必要なツールをインスールする：  
(0)rustツールのインストール
```bash

curl https://sh.rustup.rs -sSf | sh
```

(1)Cコンパイラのバージョンアップ   
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

(2)その他の必要ツールのインストール
```bash

sudo apt-get install openocd
sudo apt-get install picocom

```
## 事前準備
openocdとgdbを使用してビルドしたelfを自動的に書き込むので
以下のように起動時の.gdbinitを用意する：

(1)~/.gdbinit  
以下の内容に編集する：
```
add-auto-load-safe-path /home/user/rust_ws/mb_work/microbit/.gdbinit
```
上の「/home/user/rust_ws/mb_work/microbit/」は、gdb実行時のカレント・ディレクトリになるので、自分の環境に合わせる。

(2)~/rust_ws/mb_work/microbit/.gdbinit  
以下の内容に編集する： 
```

target remote :3333
monitor arm semihosting enable	
load
continue
```

(3)別の端末でopenocdを起動しておく。   
micro:bitをホストPCと接続して
以下を実行する。   
(gdbの接続待ちになる)
```bash

openocd -f interface/cmsis-dap.cfg -f target/nrf51.cfg
```

シリアルを利用したサンプルを実行する場合、
さらに別端末でシリアル通信を起動しておく。
```bash

picocom /dev/ttyACM0 -b115200
```

## samplesを実行する
以下を実行する：
```bash

rustup update
rustup target add thumbv6m-none-eabi

#サンプルをgitでダウンロードする
git clone https://github.com/therealprof/microbit.git
cd microbit
 
#たとえば、以下のうち一つを実行すると
#gdbが自動的に起動しプログラムを書き込んで実行される
cargo run --example gpio_hal_blinky
cargo run --example serial_direct_helloworld
cargo run --example gpio_hal_ledbutton
cargo run --example gpio_hal_printbuttons
cargo run --example led_rtfm
cargo run --example serial_hal_blocking_echo

#以下、gdbの出力例：
..
semihosting is enabled
Loading section .vector_table, size 0xa8 lma 0x0
Loading section .text, size 0x486c lma 0xa8
Loading section .rodata, size 0xc14 lma 0x4920
Start address 0x4768, load size 21800

#この時点で、書き込んだプログラムが実行される

```
なお、gdbから抜ける場合は、quitを実行する。

## 実行結果

1.gpio_hal_blinky    
micro:bitのLEDの一つが点滅する。

2.gpio_hal_ledbutton   
ボタンを押すとLEDが光る。

3.led_rtfm   
LEDがハートマークに光る。

4.serial_direct_helloworld   
シリアル出力で'hello world'が出力される。

5.serial_hal_blocking_echo   
エコープログラムになっており
シリアル通信の画面で入力すると
入力した文字が返ってくる。

6.gpio_hal_printbuttons   
ボタンを押すと、その状態を示す
テキストがシリアル出力される。

なお、上のものは、色々あるサンプルのうちの一部である。

## 参照情報

[micro:bit](https://www.switch-science.com/catalog/5263/)  

[The Embedded Rust Book](https://rust-embedded.github.io/book/intro/index.html)  
[The Embedded Rust Book - Semihosting](https://rust-embedded.github.io/book/start/semihosting.html)  

[UbuntuにRustをインストールする](https://qiita.com/yoshiyasu1111/items/0c3d08658560d4b91431)   

以上
　