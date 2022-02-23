
2020/4/19+

GO/TinyGO MODULES
# GO/TinyGO MODULE

## 概要
「go modules」の使い方について説明する。
今までは、GO(TinyGOを含む)のプログラムを置くディレクトリは
$GOPATH/src以下に限定されていたが、「go modules」を使うことにより、
それ以外のディレクトリにプログラムが置けるようになる。以下で、そのやり方について説明する。

## 準備
GOが「go modules」をサポートしているバージョンかどうか確認するために
以下を実行し、helpの本文が表示されれば該当のバージョンとなる：
```bash

go help modules

```

## 手順
```bash

# $GOPATH/srcの外側にディレクトリを作る
mkdir /tmp/hello
cd /temp/hello

# 以下を実行して、go.modファイルを作る
go mod init hello
# 「hello」を任意のようだが、通常、ディレクトリの名前にする

# 以下の実行で、go.modの内容を確認する
cat go.mod
#出力：
module hello

go 1.14
# 定義したモジュール名とgoのバージョンが登録されていることが分かる

nano hello.go
# 以下の内容のファイルを作成する：
```
hello.go
```go

package main 

import "rsc.io/quote"

func main() {
	println(quote.Glass())
}
```
続き：
```bash

# ビルドする
go build .
# 出力内容:必要なモジュールが検索されていることが分かる
go: finding module for package rsc.io/quote
go: found rsc.io/quote in rsc.io/quote v1.5.2

# 実行する
./hello
I can eat glass and it doesn't hurt me.

# 以下を実行して、go.modの内容を再度、確認する
cat go.mod
#出力：
module hello

go 1.14

require rsc.io/quote v1.5.2
# ビルドで検索されたモジュールが新たに登録されたことが分かる

# 以下を実行することで、使われているモジュールの詳細を知ることができる
go list -m all
# 出力：
hello
golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
rsc.io/quote v1.5.2
rsc.io/sampler v1.3.0

# 検索され登録されたモジュールは以下に自動的にダウンロードされる：
ls $GOPATH/pkg/mod
cache  github.com  golang.org  rsc.io  tinygo.org

```
以上で、「go modules」により$GOPATH/srcの外側のディレクトリに
プログラムが置けて、ビルド&実行できる。

## TinyGOの場合
TinyGOでも同様に「go modules」を
以下のように使用できる：
```bash

mkdir board_uno
cd board_uno
mkdir adc0
cd adc0

go mod init adc0

nano adc0.go
#プログラムを作成する。

# build&flash(arduino-unoの場合)
tinygo flash -port /dev/ttyACM0 -target=arduino .

# printlnのために通信ソフトを起動する
picocom /dev/ttyACM0 -b19200

```

## docker/tinygoの場合
カレント・ディレクトリで以下を実行することでbuild&flashができる：
```bash

docker run -it --rm -v $PWD:/go/src/github.com/myusr/myapp -w /go/src/github.com/myusr/myapp -e GOPATH=/go tinygo/tinygo:latest tinygo build -o uno.hex -target arduino .
avrdude -p atmega328p -C ~/.platformio/packages/tool-avrdude/avrdude.conf -c arduino -b 115200 -D -P /dev/ttyACM0 -U flash:w:uno.hex:i

```
ただし、「go modules」の仕掛けでの動作ではなく、ホスト環境をdocker環境にマップする際にdocker環境の$GOPATH/src以下にビルドしたいソースを置くことによりビルド可能にしている。(dockerに関する知識が不足しているので誤解している可能性もある)


## 引用情報

[go modules have landed](https://groups.google.com/forum/#!msg/golang-dev/a5PqQuBljF4/61QK4JdtBgAJ)  
[docker runのオプションいろいろ](https://scrapbox.io/llminatoll/docker_run%E3%81%AE%E3%82%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%84%E3%82%8D%E3%81%84%E3%82%8D)   


以上

