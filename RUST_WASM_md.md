
2020/4/18  

Rust WASM
# Rust WASM

## 概要
Rustの出力したwasmを動かす。

## 準備
Rustのインストール
(インストール済みであれば不要)
```bash

curl https://sh.rustup.rs -sSf | sh
rustup update

```

## 手順1
```bash

# WebAssemblyへのコンパイル機能を有効化
rustup target add wasm32-unknown-unknown

mkdir rust_wasm
cd rust_wasm

# 新しいプロジェクト wasam-test　を作成する
cargo new --lib wasm-test

nano Cargo.toml
# Cargo.tomlを編集して以下を追加する：
```
```
[lib]
crate-type = ["cdylib"]
```
続き
```bash
nano src/lib.rs
# src/lib.rs を編集して以下の内容にする：
```
src/lib.rs
```rust
#[no_mangle]
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```
続き：
```bash

# 以下を実行してビルドする
cargo build --target=wasm32-unknown-unknown --release

# 以下を実行して wasm ができていることを確認する
ls  target/wasm32-unknown-unknown/release/*.wasm
target/wasm32-unknown-unknown/release/wasm_test.wasm
# 注意：「wasm-test」が「wasm_test」になるので注意すること

nano index.html
# 以下の内容のindex.htmlを作成する：
```
index.html
```html
<!DOCTYPE html>
<html lang="ja">

<head>
  <meta charset="UTF-8">
  <title>Hello, WebAssembly!</title>
  <script>
    const wasm = './target/wasm32-unknown-unknown/release/wasm_test.wasm'
    fetch(wasm)
      .then(response => response.arrayBuffer())
      .then(bytes => WebAssembly.instantiate(bytes, {}))
      .then(results => {
        console.log('wasm output:')
        console.log(results.instance.exports.add(34, 56))
      })
  </script>
</head>
</html>
```
続き：
```bash
# http-serverを起動する
python3 -m http.server 8080

# web-browserで「localhost:8080」にアクセスする。
# consoleに以下が表示されていればＯＫ：
#  wasm output:
#  90
```
consoleに以下のようなエラーが出た場合、MIME-typeにwasmが登録されていないことになる。
```
TypeError: Failed to execute 'compile' on 'WebAssembly': Incorrect response MIME type. Expected 'application/wasm'.
```
そのときは、/etc/mime.typesにwasmのMIMEtypeを登録する。
```bash

sudo leafpad /etc/mime.types 

以下を末尾に追加する：
```
```
application/wasm                                wasm 
```
登録後、http-serverを再起動する。

## 手順２
上の手順１の方法では文字列などをJavascript側に渡すことができない。
文字列を渡したい場合は、変換ツール(wasm-bindgen-cli)によってwasmをjsのモジュールに変換し
そのモジュールを使うことにより文字列を引き渡すことができる。

以下、そのやり方について記する；

```bash

# 変換ツールのインストール
cargo install wasm-bindgen-cli

# 新しいプロジェクト wasam-test2　を作成する
cargo new --lib wasm-test

nano Cargo.toml
# Cargo.tomlを編集して以下を追加する：
```
```
[dependencies]
wasm-bindgen = "0.2"

[lib]
crate-type = ["cdylib"]
```
続き：
```bash
nano src/lib.rs
# src/lib.rs を編集して以下の内容にする：
```
src/lib.rs
```rust

extern crate wasm_bindgen;

use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn hello() -> String {
    //"Hello WASM world!".to_string() ←こちらでも良い
    String::from("Hello WASM world!")
}
#[wasm_bindgen]
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```
続き：
```bash
# 以下を実行してビルドする
cargo build --target=wasm32-unknown-unknown --release
# 以下を実行して、wasmをjsに変換する
wasm-bindgen target/wasm32-unknown-unknown/release/wasm_test2.wasm --out-dir ./pkg --target web

# jsが生成されていることの確認
ls pkg/*.js
pkg/wasm_test2.js

nano index.html
# 以下の内容のindex.htmlを作成する：
```
index.html
```html
<html>
  <head>
    <meta content="text/html;charset=utf-8" http-equiv="Content-Type"/>
  </head>
  <body>
    <script type="module" src='./index.js'></script>
    Rust -> Wasm -> Js test
  </body>
</html>
```
注意：「<script type="module" 」であることに注意のこと

```bash
nano index.js
# 以下の内容を作成する：
```
index.js
```javascript

// in script[type=module]
import init, {add, hello } from './pkg/wasm_test2.js';

(async () => {
  await init();

  // call Rust func
  console.log(hello());
  console.log('wasm->js output:')
  console.log(add(34,56));
})();
```
続き：
```bash
# http-serverを起動する
python3 -m http.server 8080

# web-browserで「localhost:8080」にアクセスする。
# consoleに以下が表示されていればＯＫ：
#  Hello WASM world!
#  wasm->js output:
#  90

```

## サイズ情報
```bash
$ ls -l target/wasm32-unknown-unknown/release/wasm_test2.wasm
-rwxrwxr-x 2 xxxx 1794765  4月 18 10:49 target/wasm32-unknown-unknown/release/wasm_test2.wasm

$ ls -l pkg
合計 32
-rw-rw-r-- 1 xxxx   881  4月 18 10:49 wasm_test2.d.ts
-rw-rw-r-- 1 xxxx  2831  4月 18 10:49 wasm_test2.js
-rw-rw-r-- 1 xxxx   235  4月 18 10:49 wasm_test2_bg.d.ts
-rw-rw-r-- 1 xxxx 16946  4月 18 10:49 wasm_test2_bg.wasm

```
wasm_test2.wasmと(変換後の)wasm_test2.jsのサイズを比較すると、ずいぶん小さくなっている印象があるが、該当モジュールのなかで、wasm_test2_bg.wasmを呼び出しているようだ。

## 参考情報

[GO/TinyGOの出力したwasmを動かす](https://beta-notes.way-nifty.com/blog/2020/04/post-4897c6.html)  

[WebAssembly入門](https://wasm-dev-book.netlify.app/hello-wasm.html)  
[バンドラを使わずにRustをWASMにする](https://lealog.hateblo.jp/entry/2019/12/27/105424)  
[爆速でjsでWebAssemblyを動かす](https://qiita.com/mk-tool/items/eb537ad0ce9639f58e1f) 

[Incorrect response MIME type. Expected 'application/wasm'エラーの対応](https://qiita.com/kamykn/items/0bd8b1dfb342ab478e4a)  


以上

