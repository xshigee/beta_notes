
2020/5/8+

Rust STM32F3xx
# Rust STM32F3xx

## 概要
プログラミング言語Rustを以下のSTM32F3Discovery/Nucleo-F303K8ボードで動かす。
(ホストPCとしてはubuntuを想定している)

[STM32F3DISCOVERY](https://www.st.com/en/evaluation-tools/stm32f3discovery.html)   
[STM32 Nucleo Board STM32F303K8](http://akizukidenshi.com/catalog/g/gM-10172/)   

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


## sample#0(usb_serial)をビルドする
最初の設定がターゲットボードとしてSTM32F3Discoveryになっているので、まずは、STM32F3Discoveryで動かす。

```bash

git clone https://github.com/stm32-rs/stm32f3xx-hal.git
cd stm32f3xx-hal

rustup target add thumbv7em-none-eabihf

cargo clean
cargo build --example usb_serial --release --features="rt stm32f303xc stm32-usbd"

```

## ビルドしたfirmwareを書き込む
```bash

# elfをbinに変換する
arm-none-eabi-objcopy -O binary target/thumbv7em-none-eabihf/release/examples/usb_serial firmware.bin

# binを書き込む
#[USB ST-LINK]と印刷したあるUSBコネクタをホストPCに接続する
st-flash write firmware.bin 0x8000000
出力例：
$ st-flash write firmware.bin 0x8000000
st-flash 1.6.0
2020-05-08T09:42:28 INFO usb.c: -- exit_dfu_mode
2020-05-08T09:42:28 INFO common.c: Loading device parameters....
2020-05-08T09:42:28 INFO common.c: Device connected is: F3 device, id 0x10036422
2020-05-08T09:42:28 INFO common.c: SRAM size: 0xa000 bytes (40 KiB), Flash: 0x40000 bytes (256 KiB) in pages of 2048 bytes
2020-05-08T09:42:28 INFO common.c: Attempting to write 17148 (0x42fc) bytes to stm32 address: 134217728 (0x8000000)
Flash page at addr: 0x08004000 erased
2020-05-08T09:42:29 INFO common.c: Finished erasing 9 pages of 2048 (0x800) bytes
2020-05-08T09:42:29 INFO common.c: Starting Flash write for VL/F0/F3/F1_XL core id
2020-05-08T09:42:29 INFO flash_loader.c: Successfully loaded flash loader in sram
  9/9 pages written
2020-05-08T09:42:30 INFO common.c: Starting verification of write complete
2020-05-08T09:42:30 INFO common.c: Flash written and verified! jolly good!


#USBシリアルのデモプログラムなので、[USB USER]と印刷してあるUSBコネクタをホストＰＣに接続する
picocom /dev/ttyACM0 -b115200
#エコープログラムなので、キーボードから入力し、そのまま文字が表示される。
#(ただし、英字はシフトに関係なく、全て大文字になる)
#以上のような動きになればＯＫとなる

#以下でUSBデバイスの状況を確認する
dmesg
出力例：
<省略>
[38985.161676] usb 1-1.2: Product: Serial port
[38985.161680] usb 1-1.2: Manufacturer: Rust Embedded company
[38985.161683] usb 1-1.2: SerialNumber: STM32F3Disco
[38985.162359] cdc_acm 1-1.2:1.0: ttyACM0: USB ACM device
# 以上のように、Product/Manufacturer/SerialNumberが返ってくることが確認できる
# ただし、上の例では、コンパイルの確認のためにManufacturer/SerialNumberを変更している

```
## sample#1(toggle)
以下の手順で別のサンプルを実行できる：
```bash

cargo clean
cargo build --example toggle --release --features="rt stm32f303xc"
arm-none-eabi-objcopy -O binary target/thumbv7em-none-eabihf/release/examples/toggle firmware.bin
#[USB ST-LINK]と印刷してあるUSBコネクタをホストPCに接続する
st-flash write firmware.bin 0x8000000

```
ボードのLD10が点滅すれば動作ＯＫとなる。

なお、以下の対応関係があるので、examples/toggle.rsの「.pe13」の部分を
別のポートに変更すれば、他のＬＥＤを点滅させることができる。
```
User LD3: red LED: PE9 of the STM32F303VCT6.
User LD4: blue LED: PE8 of the STM32F303VCT6.
User LD5: orange LED: PE10 of the STM32F303VCT6.
User LD6: green LED: PE15 of the STM32F303VCT6.
User LD7: green LED: PE11 of the STM32F303VCT6.
User LD8: orange LED: PE14 of the STM32F303VCT6.
User LD9: blue LED: PE12 of the STM32F303VCT6.
User LD10: red LED: PE13 of the STM32F303VCT6.
```
## sample#2(pwm)
以下の手順で別のサンプルを実行できる：
```bash

cargo clean
cargo build --example pwm --release --features="stm32f303xc"
arm-none-eabi-objcopy -O binary target/thumbv7em-none-eabihf/release/examples/pwm firmware.bin
#[USB ST-LINK]と印刷してあるUSBコネクタをホストPCに接続する
st-flash write firmware.bin 0x8000000

```
PA4,PA7,PB5に接続したLEDがPWMで光れば動作としてはＯＫとなる。

## OpenOCDによる書き込み
今までは、st-flashで書き込んだが、以下の手順でOpenOCDで書き込むこともできる。   
別の端末で以下を実行する：
```bash

openocd -f interface/stlink-v2.cfg -f target/stm32f3x.cfg
```
エラーになった場合、stlinkのファームウェアが新しい可能性があるので
そのときは「tlink-v2.cfg」を「tlink-v2-1.cfg」に変更して実行する。
実行するとopenocdがサーバーとして動作して、gdb(やtelnet)を待ち受ける状態になる。

buildの作業で使っている端末で以下を実行する：
(usb_serialの場合)
```bash

cargo run --example usb_serial --release --features="rt stm32f303xc stm32-usbd"
#ビルドが完了すると自動的にgdbが起動するので
#以下の手順でopenocdと接続してファームウェアを書き込む。
(gdb) target remote :3333
Remote debugging using :3333
0x00000000 in ?? ()
(gdb) load
Loading section .vector_table, size 0x194 lma 0x8000000
Loading section .text, size 0x34f4 lma 0x8000194
Loading section .rodata, size 0xc6c lma 0x8003688
Start address 0x8000e8e, load size 17140
Transfer rate: 15 KB/sec, 5713 bytes/write.
(gdb) quit
```
openocdの起動用スクリプトを設定すると書き込みを自動化できるようだが、ここでは、それを使用していない。

## memory.x
stm32f3xx-halディレクトリにあるmemory.xはリンク用スクリプトで以下のように設定されている:
```
/* Linker script for the STM32F303VCT6 */
MEMORY
{
  CCRAM : ORIGIN = 0x10000000, LENGTH = 8K
  FLASH : ORIGIN = 0x08000000, LENGTH = 256K
  RAM : ORIGIN = 0x20000000, LENGTH = 40K
}

_stack_start = ORIGIN(CCRAM) + LENGTH(CCRAM);
```
これを他のボード(F303xxxx)用に変更すれば、
他のボードでもビルドできるはずである。

## ターゲット情報

Cortex-Mプロセッサとターゲットは以下の対応になっている：
```

thumbv6m-none-eabi - Cortex-M0 と M0+
thumbv7m-none-eabi - Cortex M3
thumbv7em-none-eabi - Cortex M4 と M7、FPU なし
thumbv7em-none-eabihf - Cortex M4 と M7、FPU あり

```

## Nucleo-F303K8対応
ここからは、設定をNucleo-F303K8に変更して、Nucleo-F303K8で動かしてみる。  
そのために以下の変更/修正を行なう：   
(1)memory.xの変更   
以下のように変更する：
```


/* Linker script for the STM32F303K8 */
MEMORY
{
  FLASH : ORIGIN = 0x08000000, LENGTH = 64K
  RAM : ORIGIN = 0x20000000, LENGTH = 12K
}

_stack_start = ORIGIN(RAM) + LENGTH(RAM);
```
(2)example/toggle.rsの修正   
(ＬＥＤのポートを変更する)
```rust

20行付近：
    let mut gpioe = dp.GPIOE.split(&mut rcc.ahb);
→
    let mut gpiob = dp.GPIOB.split(&mut rcc.ahb);
```
```rust

25行付近：
    let mut led = gpioe
        .pe13
        .into_push_pull_output(&mut gpioe.moder, &mut gpioe.otyper);
→
    let mut led = gpiob //gpioe
        .pb3
        .into_push_pull_output(&mut gpiob.moder, &mut gpiob.otyper);


```
(3)ビルド＆書き込み＆実行
```bash

cargo clean
cargo build --example toggle --release --features="rt stm32f303xc"
arm-none-eabi-objcopy -O binary target/thumbv7em-none-eabihf/release/examples/toggle firmware.bin
#F303K8のUSBコネクタをホストPCに接続する
st-flash write firmware.bin 0x8000000
```
F303K8ボードのグリーンLEDが点滅すれば動作としてはＯＫとなる。


## 参照情報

[STM32F3DISCOVERY](https://www.st.com/en/evaluation-tools/stm32f3discovery.html)   
[STM32 Nucleo Board STM32F303K8](http://akizukidenshi.com/catalog/g/gM-10172/)   

[The Embedded Rust Book](https://rust-embedded.github.io/book/intro/index.html)  
[The Embedded Rust Book - Semihosting](https://rust-embedded.github.io/book/start/semihosting.html)  

[UbuntuにRustをインストールする](https://qiita.com/yoshiyasu1111/items/0c3d08658560d4b91431)   

以上
