
2020/5/11

Rust STM32F1xx
# Rust STM32F1xx

## 概要
プログラミング言語Rustを以下のNucleo-F103RBボードで動かす。
(ホストPCとしてはubuntuを想定している)

[STM32 Nucleo Board STM32F103RB](http://akizukidenshi.com/catalog/g/gM-07724/)   

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


## sample#0(blinky)を実行する
最初の設定のターゲットボードがblue-pillになっているので、Nucleo-F103RBの設定に変更して動かす。

(1)rust関連の設定
```bash

git clone https://github.com/stm32-rs/stm32f1xx-hal.git
cd stm32f1xx-hal

rustup default nightly

rustup target add thumbv7m-none-eabi

```

(2)memory.xの変更   
以下のように変更する：  
(Flash容量を64Kから128Kに変更する)
```
cat memory.x 
/* Linker script for the NUCLEO-F103RB */
MEMORY
{
  FLASH : ORIGIN = 0x08000000, LENGTH = 128K
  RAM : ORIGIN = 0x20000000, LENGTH = 20K
}
```

(3)examples/blinky.rsの変更  
(オンボードのLEDのポートをpc13からpa5に変更する)
```rust

37行付近：
    let mut gpioc = dp.GPIOC.split(&mut rcc.apb2);
→
    let mut gpioa = dp.GPIOA.split(&mut rcc.apb2);
```
```rust

40行付近：
    let mut led = gpioc.pc13.into_push_pull_output(&mut gpioc.crh);
→
    let mut led = gpioa.pa5.into_push_pull_output(&mut gpioa.crl);

```
点滅周期を変更したい場合、以下のような変更を行なう：
```rust

42行付近：
    let mut timer = Timer::syst(cp.SYST, &clocks).start_count_down(1.hz());
→
    let mut timer = Timer::syst(cp.SYST, &clocks).start_count_down(10.hz());
```

(4)ビルド
```bash

cargo clean
cargo build --example blinky --release --features=stm32f103
```

(5)elfをbinに変換する
```bash

arm-none-eabi-objcopy -O binary target/thumbv7m-none-eabi/release/examples/blinky F103_blinky.bin
```

(6)書き込む
```bash

#ボートとホストPCをUSBで接続する
st-flash write F103_blinky.bin 0x8000000
```
ボードのグリーンLED(LD2)が点滅すれば動作としてはＯＫとなる。

## その他のサンプル
以下に、その他のサンプルがあるのでボードに依存するポートなどを変更すれば、Nucleo-F103RBで動作すると思われる。
```bash

$ ls examples

adc-dma-circ.rs            itm.rs               serial-dma-peek.rs
adc-dma-rx.rs              led.rs               serial-dma-rx.rs
adc.rs                     mfrc522.rs           serial-dma-tx.rs
adc_temperature.rs         motor.rs.disabled    serial-fmt.rs
blinky.rs                  mpu9250.rs.disabled  serial.rs
blinky_generic.rs          nojtag.rs            serial_config.rs
blinky_rtc.rs              panics.rs            spi-dma.rs
blinky_timer_irq.rs        pwm.rs               timer-interrupt-rtfm.rs
delay.rs                   pwm_custom.rs        usb_serial.rs
enc28j60-coap.rs.disabled  pwm_input.rs         usb_serial_interrupt.rs
enc28j60.rs.disabled       qei.rs               usb_serial_rtfm.rs
exti.rs                    rtc.rs
hello.rs                   serial-dma-circ.rs

```

## ターゲット情報

Cortex-Mプロセッサとターゲットは以下の対応になっている：
```

thumbv6m-none-eabi - Cortex-M0 と M0+
thumbv7m-none-eabi - Cortex M3
thumbv7em-none-eabi - Cortex M4 と M7、FPU なし
thumbv7em-none-eabihf - Cortex M4 と M7、FPU あり

```

## 参照情報

[STM32 Nucleo Board STM32F103RB](http://akizukidenshi.com/catalog/g/gM-07724/)   

[The Embedded Rust Book](https://rust-embedded.github.io/book/intro/index.html)  
[The Embedded Rust Book - Semihosting](https://rust-embedded.github.io/book/start/semihosting.html)  

[UbuntuにRustをインストールする](https://qiita.com/yoshiyasu1111/items/0c3d08658560d4b91431)   

以上
