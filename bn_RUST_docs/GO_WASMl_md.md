2020/4/18  
Dockerでのビルド方法を追加した  
2020/4/17  
初版   

GO/TinyGO WASM
# GO/TinyGO WASM

## 概要
GO/TinyGOの出力したwasmを動かす。

## 手順
```bash

cd wasm
mkdir go-wasm
cd go-wasm
mkdir out
mkdir go

nano go/main.go
#以下の内容になるよう編集する：
```
go/main.go
```go
package main

func main() {
    println("Hello TinyGO World!!!")
}
```
続き：
```bash

# GOでwasmを出力する場合、以下を実行する(GO/TinyGOで排他的使用)：
GOOS=js GOARCH=wasm go build -o out/main.wasm go/main.go

# TinyGOでwasmを出力する場合、以下を実行する(GO/TinyGOで排他的使用)：
tinygo build -o out/main.wasm -target wasm ./go/main.go

# Docker/TinyGOでwasmを出力する場合、以下を実行する(GO/TinyGOで排他的使用)：
docker run --rm -v $(pwd):/src -v $GOPATH:/go -e "GOPATH=/go" tinygo/tinygo:0.12.0 tinygo build  -o /src/out/main.wasm -target=wasm wasm/go-wasm/go
#「wasm/go-wasm/go」の部分は自分の環境にあわせること
# $GOPATH/src以下のmain.goがあるディレクトリまでのパスを指定すること
# 上の例では $GOPATH/src/wasm/go-wasm/go を指定したことになる

# alias設定したDocker/TinyGOでwasmを出力する場合、以下を実行する(GO/TinyGOで排他的使用)：
tigo build -o /src/out/main.wasm -target=wasm wasm/go-wasm/go

cd out
# wasmの実行に必要なファイルをダウンロードする：

# GOのwasmを実行する場合は、以下のファイルをダウンロードする(GO/TinyGOで排他的使用)
wget https://raw.githubusercontent.com/golang/go/master/misc/wasm/wasm_exec.js

# TinyGOのwasmを実行する場合は、以下のファイルをダウンロードする(GO/TinyGOで排他的使用)
wget https://raw.githubusercontent.com/tinygo-org/tinygo/master/targets/wasm_exec.js

# 以下のhtmlファイルをダウンロードする(GO/TinyGO共通)
wget https://raw.githubusercontent.com/golang/go/master/misc/wasm/wasm_exec.html
cp wasm_exec.html index.html
nano index.html
#index.htmlの内容の「"test.wasm"」を「"main.wasm"」に変更する

# http-serverを起動する
python3 -m http.server 8080

# web-browserで「localhost:8080」にアクセスする。
# web画面の[Run]を押してweb-consoleに入る。
# consoleに以下が表示されていればＯＫ：
#  Console was cleared
#  Hello TinyGO World!!!
 
```
consoleに以下のようなエラーが出た場合、MIME-typeにwasmが登録されていないことになる。
```
TypeError: Failed to execute 'compile' on 'WebAssembly': Incorrect response MIME type. Expected 'application/wasm'.
```
そのときは、/etc/mime.typesにwasmのMIMEtypeを登録する。
```bash

sudo leafpad /etc/mime.types 

以下を末尾に追加する；
application/wasm                                wasm 
```
登録後、http-serverを再起動する。

## GOとTinyGOの出力したwasmのサイズ比較
GO/TinyGOのwasmのサイズを比較すると以下のような状況だった：
```bash
$ ls -l
-rwxrwxr-x 1 xxxx 1252157  4月 17 11:18 GO_main.wasm
-rwxrwxr-x 1 xxxx    4729  4月 17 11:24 TinyGO_main.wasm
-rw-rw-r-- 1 xxxx    1303  4月 17 09:29 index.html
-rwxrwxr-x 1 xxxx    4729  4月 17 11:19 main.wasm
-rw-rw-r-- 1 xxxx    1303  4月 17 09:29 wasm_exec.html
-rw-rw-r-- 1 xxxx   17390  4月 17 09:29 wasm_exec.js

```
ずいぶんサイズに差があるので、TinyGOのほうが
webで使用する際にはロード時間などの高速化が期待できそうである。

## 引用情報

[(Tiny)Go to WebAssembly](https://dev.to/sendilkumarn/tiny-go-to-webassembly-5168)   
[コンピュータボードでTinyGOを動かす](https://beta-notes.way-nifty.com/blog/2020/04/post-1ecc01.html)  

[Incorrect response MIME type. Expected 'application/wasm'エラーの対応](https://qiita.com/kamykn/items/0bd8b1dfb342ab478e4a)  


以上

