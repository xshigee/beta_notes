
2020/4/9

Rust Install OSC
# Rust Install OSC

## 概要
プログラミング言語RustでOSC(Open Sound Control)を動かす。
ここでは、開発ツールのインストールから説明する。実行するプログラムはOSCライブラリのexamplesを流用している。
(ホストPCとしてはubuntuを想定している)

## 開発ツールのインストール
```bash

mkdir rust_ws
cd rust_ws

curl https://sh.rustup.rs -sSf | sh

```
ツールの動作確認：
```bash

# プロジェクトhello_cargo を作る
cargo new hello_cargo --bin
cd hello_cargo
# build&run
cargo build
cargo run
# 以下のような出力が出ればＯＫとなる：
    Finished dev [unoptimized + debuginfo] target(s) in 0.02s
~/rust_ws/hello_cargo$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/hello_cargo`
Hello, world!

```

## OSC受信プログラム
(1)プロジェクトを作る：
```bash

cargo new osc_recv --bin
cd osc_recv

```
(2)Cargo.tomlを編集する：  
[dependencies]に以下を追加する：
```bash

[dependencies]
rosc = "~0.3"

```

(3)src/main.rsを以下のように編集する：
```rust

extern crate rosc;

use rosc::OscPacket;
use std::env;
use std::net::{SocketAddrV4, UdpSocket};
use std::str::FromStr;

fn main() {
    let args: Vec<String> = env::args().collect();

    let usage = format!("Usage {} IP:PORT", &args[0]);
    if args.len() < 2 {
        println!("{}", usage);
        ::std::process::exit(1)
    }

    let addr = match SocketAddrV4::from_str(&args[1]) {
        Ok(addr) => addr,
        Err(_) => panic!(usage),
    };
    let sock = UdpSocket::bind(addr).unwrap();
    println!("Listening to {}", addr);

    let mut buf = [0u8; rosc::decoder::MTU];

    loop {
        match sock.recv_from(&mut buf) {
            Ok((size, addr)) => {
                println!("Received packet with size {} from: {}", size, addr);
                let packet = rosc::decoder::decode(&buf[..size]).unwrap();
                handle_packet(packet);
            }
            Err(e) => {
                println!("Error receiving from socket: {}", e);
                break;
            }
        }
    }
}

fn handle_packet(packet: OscPacket) {
    match packet {
        OscPacket::Message(msg) => {
            println!("OSC msg: {} {:?}", msg.addr, msg.args);
        }
        OscPacket::Bundle(bundle) => {
            println!("OSC Bundle: {:?}", bundle);
        }
    }
}

```

(4)build&run
```bash

cargo build
cargo run 0.0.0.0:8000

```
入力ポートとして8000として、OSC受信する。

出力例：
```bash

# TouchOSCからOSCを受信したときの出力
Received packet with size 20 from: 192.168.0.10:9000
OSC msg: /1/fader1 [Float(0.5843882)]
Received packet with size 20 from: 192.168.0.10:9000
OSC msg: /1/fader1 [Float(0.5822785)]
Received packet with size 20 from: 192.168.0.10:9000
OSC msg: /1/fader1 [Float(0.5822785)]
Received packet with size 20 from: 192.168.0.10:9000
OSC msg: /1/fader1 [Float(0.5801688)]
Received packet with size 20 from: 192.168.0.10:9000
OSC msg: /1/fader1 [Float(0.5801688)]

# 本記事のOSC_sendのOSCを受信したときの出力
Received packet with size 20 from: 127.0.0.1:9000
OSC msg: /3/xy2 [Float(0.99939775), Float(0.4754662)]
Received packet with size 20 from: 127.0.0.1:9000
OSC msg: /3/xy1 [Float(0.5), Float(1.0)]
Received packet with size 20 from: 127.0.0.1:9000
OSC msg: /3/xy2 [Float(1.0), Float(0.5)]
Received packet with size 20 from: 127.0.0.1:9000
OSC msg: /3/xy1 [Float(0.52453387), Float(0.99939775)]
Received packet with size 20 from: 127.0.0.1:9000
OSC msg: /3/xy2 [Float(0.99939775), Float(0.52453387)]

```


## OSC送信プログラム
(1)プロジェクトを作る：
```bash

cargo new osc_send --bin
cd osc_send

```
(2)Cargo.tomlを編集する：
[dependencies]に以下を追加する：
```bash

[dependencies]
rosc = "~0.3"

```

(3)src/main.rsを以下のように編集する：
```rust

use rosc::encoder;
use rosc::{OscMessage, OscPacket, OscType};
use std::net::{SocketAddrV4, UdpSocket};
use std::str::FromStr;
use std::time::Duration;
use std::{env, f32, thread};

fn get_addr_from_arg(arg: &str) -> SocketAddrV4 {
    SocketAddrV4::from_str(arg).unwrap()
}

fn main() {
    let args: Vec<String> = env::args().collect();
    let usage = format!(
        "Usage: {} HOST_IP:HOST_PORT CLIENT_IP:CLIENT_PORT",
        &args[0]
    );
    if args.len() < 3 {
        panic!(usage);
    }
    let host_addr = get_addr_from_arg(&args[1]);
    let to_addr = get_addr_from_arg(&args[2]);
    let sock = UdpSocket::bind(host_addr).unwrap();

    // switch view
    let msg_buf = encoder::encode(&OscPacket::Message(OscMessage {
        addr: "/3".to_string(),
        args: vec![],
    }))
    .unwrap();

    sock.send_to(&msg_buf, to_addr).unwrap();

    // send random values to xy fields
    let steps = 128;
    let step_size: f32 = 2.0 * f32::consts::PI / steps as f32;
    for i in 0.. {
        let x = 0.5 + (step_size * (i % steps) as f32).sin() / 2.0;
        let y = 0.5 + (step_size * (i % steps) as f32).cos() / 2.0;
        let mut msg_buf = encoder::encode(&OscPacket::Message(OscMessage {
            addr: "/3/xy1".to_string(),
            args: vec![OscType::Float(x), OscType::Float(y)],
        }))
        .unwrap();

        sock.send_to(&msg_buf, to_addr).unwrap();
        msg_buf = encoder::encode(&OscPacket::Message(OscMessage {
            addr: "/3/xy2".to_string(),
            args: vec![OscType::Float(y), OscType::Float(x)],
        }))
        .unwrap();
        sock.send_to(&msg_buf, to_addr).unwrap();
        thread::sleep(Duration::from_millis(20));
    }
}

```


(4)build&run
```bash

cargo build
cargo run 0.0.0.0:9000 127.0.0.1:8000
```
出力ポートを8000に設定してOSC送信する。

出力例：
```bash
$ cargo run 0.0.0.0:9000 127.0.0.1:8000
    Finished dev [unoptimized + debuginfo] target(s) in 0.03s
     Running `target/debug/osc_send '0.0.0.0:9000' '127.0.0.1:8000'`

```

## 参照情報

[rosc is an implementation of the OSC 1.0 protocol in pure Rust.](https://github.com/klingtnet/rosc)   

[UbuntuにRustをインストールする](https://qiita.com/yoshiyasu1111/items/0c3d08658560d4b91431)   

[TouchOSC](https://hexler.net/products/touchosc)  
[TouchOSC iPhone](https://apps.apple.com/app/touchosc/id288120394)  


以上
