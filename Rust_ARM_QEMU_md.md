
2020/4/12

Rust ARM QEMU
# Rust ARM QEMU

## 概要
プログラミング言語RustでARMを動かす。
実際のボードではないがQEMUのARMのRustプログラムを動かす方法について記する。
(ホストPCとしてはubuntuを想定している)

## 関連ツールのインストール
rustの開発ツールがインストール済みの前提で
追加で以下のインストールを行なう：
```bash

sudo apt install gdb-multiarch openocd qemu-system-arm

rustup target add thumbv6m-none-eabi thumbv7m-none-eabi thumbv7em-none-eabihf riscV32imac-unknown-none-elf 

cargo install cargo-generate
cargo install cargo-binutils
rustup component add llvm-tools-preview

```

## プロジェクト cortex-m を作る
```
cargo generate --git https://github.com/rust-embedded/cortex-m-quickstart
プロジェクト名を聞かれるので以下を入力する：(任意)
cortex-m

```
## 設定ならびにソース変更
```bash

cd cortex-m

nano .cargo/config 
「# 」の行のコメントを外し
「runner = ...」 にする。

nano　src/main.rs
既にあるソースを以下のように変更する。
```

src/main.rs  
(実際の変更は数行のみ)
```rust

#![no_std]
#![no_main]

// pick a panicking behavior
extern crate panic_halt; // you can put a breakpoint on `rust_begin_unwind` to catch panics
// extern crate panic_abort; // requires nightly
// extern crate panic_itm; // logs messages over ITM; requires ITM support
// extern crate panic_semihosting; // logs messages to the host stderr; requires a debugger

use cortex_m::asm;
use cortex_m_rt::entry;
use cortex_m_semihosting::{debug, hprintln};

#[entry]
fn main() -> ! {
    hprintln!("Hello Embedded World!").unwrap();

    asm::nop(); // To not have main optimize to abort in release mode, remove when you add code

    debug::exit(debug::EXIT_SUCCESS);

    loop {
        // your code goes here
    }
}

```
## build&run
```bash

cargo build
cargo run
#出力例：
    Finished dev [unoptimized + debuginfo] target(s) in 0.04s
     Running `qemu-system-arm -cpu cortex-m3 -machine lm3s6965evb -nographic -semihosting-config enable=on,target=native -kernel target/thumbv7m-none-eabi/debug/cortex-m`
Hello Embedded World!

```

## 実行バイナリーの場所
実行バイナリーは以下のある：
```bash

$ ls -l target/thumbv7m-none-eabi/debug/cortex-m
-rwxrwxr-x 2 komatsu komatsu 765276  4月 12 12:37 target/thumbv7m-none-eabi/debug/cortex-m

$ file target/thumbv7m-none-eabi/debug/cortex-m
target/thumbv7m-none-eabi/debug/cortex-m: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked, not stripped
#プロジェクト名が実行バイナリーの名前になる

```
なお、ターゲットCPUなどを変更する場合は、 .cargo/config　を編集することになる。


## 参照情報

[The Embedded Rust Book](https://rust-embedded.github.io/book/intro/index.html)  
[The Embedded Rust Book - Semihosting](https://rust-embedded.github.io/book/start/semihosting.html)  

[UbuntuにRustをインストールする](https://qiita.com/yoshiyasu1111/items/0c3d08658560d4b91431)   

以上
