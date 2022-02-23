
2020/9/27:  
シンプルなblinkを追加した。

2020/9/21:  
他のサンプルを追加した。

2020/9/20:   
ネットワーク対応サンプルを追加した。

2020/9/19+:  
初版  

Moddable install
# Moddable install

## 概要
以下のModdableをインストールする。   
ボードコンピュータでJavascriptを動かす環境としてModdableがあるが、それのインストールについて記する。   
　組み込み系ソフトとしてJavascriptが動作する例として、Fitbit_Versaなどがあるが、これは、簡単な言い方をすると、
画面関係はSVG、ロジックはJavascriptで作成できる。JavascriptのインタープリタはVersa本体に組み込まれていて、
プログラムしたJavascriptのソースをVersa本体に書き込んで実行することになる。(ソース自身はコンパクトな形に変換される)  
　obnizでもJavascriptが利用できるが、サーバークライアント・ベースでWebブラウザーでJavascriptが動作して、obnizのハードウェア自身はIOプロセッサ的に動作する。  
　TesselボードでもJavascriptが動作する。ただし、特定のハードウェアに限定される。   
　これに対して、moddableでは、Javascriptの機能自身が書き込む実行形式に含まれるので、これのみで完結する。

[moddable](https://www.moddable.com/)  

## install(シミュレータ)
以下の手順でツールをインストールする：
```bash

sudo apt-get install libgtk-3-dev

mkdir ~/Projects
#(Projectsは任意)
cd ~/Projects
git clone https://github.com/Moddable-OpenSource/moddable

export MODDABLE=~/Projects/moddable

# Moddableのコマンドラインツール(mcconfig),シミュレータ(simulator),デバッガ(xsbug)をビルドする

cd $MODDABLE/build/makefiles/lin
make
make install

export PATH=$PATH:$MODDABLE/build/bin/lin/release

# デバッグ起動テスト
xsbug

# ここでxsbugのウィンドウが立ち上がる。
# アプリケーションと接続していないので、空の画面になる。

# サンプルをコンパイル＆実行する
cd $MODDABLE/examples/piu/balls
mcconfig -d -m -p lin

# ここで、M5Stick-Cのシミュレータが動作して画面にバウンドしているボールが表示される。
# mcconfigがビルド用のツールになる；
# オプションの意味は以下のとおり：
#  -d  デバッグモードでビルド
#  -m ビルドとデバイスへの書き込みを同時に行う
#  -p ビルド対象のプラットフォームを指定するオプション。linは「LinuxのModdableシミュレータ」を指す。

```

## install(ESP32)
以下の手順でESP32用ツールをインストールする：
```bash

cd ~
mkdir esp32
cd ~/esp32
wget https://dl.espressif.com/dl/xtensa-esp32-elf-linux64-1.22.0-80-g6c4433a-5.2.0.tar.gz
tar -xvf xtensa-esp32-elf-linux64-1.22.0-80-g6c4433a-5.2.0.tar.gz
git clone -b v3.3.2 --recursive https://github.com/espressif/esp-idf.git

sudo apt-get install gcc git wget make libncurses-dev flex bison gperf python python-pip python-setuptools python-serial

export IDF_PATH=~/esp32/esp-idf

python -m pip install --user -r $IDF_PATH/requirements.txt

export PATH=$PATH:~/esp32/xtensa-esp32-elf/bin:$IDF_PATH/tools

# サンプルをコンパイル＆実行する
cd $MODDABLE/examples/piu/balls

export UPLOAD_PORT=/dev/ttyUSB0

M5Stackの場合：
mcconfig -d -m -p esp32/m5stack
# コンパイルはできるが実行はできなかった

M5Stack Fireの場合：
mcconfig -d -m -p esp32/m5stack_fire
# コンパイル実行するとデバッガ画面が表示されるので、実行アイコン(▶)を押す。

M5Stick-Cの場合：
mcconfig -d -m -p esp32/m5stick_c

M5Atomの場合：
mcconfig -d -m -p esp32/m5atom
```

## export登録
.bashrcに以下を登録する：
```

# moddable
export MODDABLE=~/Projects/moddable
export PATH=$PATH:$MODDABLE/build/bin/lin/release
export IDF_PATH=~/esp32/esp-idf
export PATH=$PATH:~/esp32/xtensa-esp32-elf/bin:$IDF_PATH/tools
```

## sample code（参考）
balls/main.js
```javascript

/*
 * Copyright (c) 2016-2020 Moddable Tech, Inc.
 *
 *   This file is part of the Moddable SDK.
 * 
 *   This work is licensed under the
 *       Creative Commons Attribution 4.0 International License.
 *   To view a copy of this license, visit
 *       <https://creativecommons.org/licenses/by/4.0>
 *   or send a letter to Creative Commons, PO Box 1866,
 *   Mountain View, CA 94042, USA.
 *
 */

import {} from "piu/MC";

const backgroundSkin = new Skin({ fill:"silver" });
const ballTexture = new Texture("balls.png");
const ballSkin = new Skin({ texture:ballTexture, x:0, y:0, width:30, height:30, variants:30 });

class BallBehavior extends Behavior {
	onCreate(ball, delta) {
		this.dx = delta;
		this.dy = delta;
	}
	onDisplaying(ball) {
		this.x = ball.x;
		this.y = ball.y;
		this.width = ball.container.width - ball.width;
		this.height = ball.container.height - ball.height;
		ball.start();
	}
	onTimeChanged(ball) {
		var dx = this.dx;
		var dy = this.dy;
		ball.moveBy(dx, dy);
		var x = this.x + dx;
		var y = this.y + dy;
		if ((x < 0) || (x > this.width)) dx = -dx;
		if ((y < 0) || (y > this.height)) dy = -dy;
		this.dx = dx;
		this.dy = dy;
		this.x = x;
		this.y = y;
	}
};

let BallApplication = Application.template($ => ({
	skin:backgroundSkin,
	contents: [
		Content(6, { left:0, top:0, skin:ballSkin, variant:0, Behavior: BallBehavior } ),
		Content(5, { right:0, top:0, skin:ballSkin, variant:1, Behavior: BallBehavior } ),
		Content(4, { right:0, bottom:0, skin:ballSkin, variant:2, Behavior: BallBehavior } ),
		Content(3, { left:0, bottom:0, skin:ballSkin, variant:3, Behavior: BallBehavior } ),
	]
}));

export default new BallApplication(null, { displayListLength:4096, touchCount:0 });
```

## ネットワーク対応のサンプル
ネットワーク対応のサンプルをビルド実行する場合、以下のように設定するssid,passwordを
コマンドラインのオプションとして追加する：
```
mcconfig -d -m -p esp32/m5atom  ssid=your_ssid password=your_passswd
```

### MQTT

ビルド実行：
```
cd $MODDABLE/examples/network/mqtt/mqttbasic

mcconfig -d -m -p esp32/m5atom  ssid=your_ssid password=your_passswd
```

xsbugのコンソール画面：
```
Wi-Fi connected to "your_ssid"
IP address 192.168.0.17
received "moddable/mqtt/example/date": Sun Sep 20 2020 18:24:23 GMT+0900
received "moddable/mqtt/example/random": 0.8646926234626893
received "moddable/mqtt/example/date": Sun Sep 20 2020 18:24:24 GMT+0900
received "moddable/mqtt/example/random": 0.3546896857104007
<省略>
received "moddable/mqtt/example/random": 0.9578066274891157
received "moddable/mqtt/example/date": Sun Sep 20 2020 18:25:34 GMT+0900
received "moddable/mqtt/example/random": 0.7666543393318203
received "moddable/mqtt/example/date": Sun Sep 20 2020 18:25:35 GMT+0900
received "moddable/mqtt/example/random": 0.3196750421355653
```

### httpserver

ビルド実行：
```
cd $MODDABLE/examples/network/http/httpserver

mcconfig -d -m -p esp32/m5atom  ssid=your_ssid password=your_passswd
```

xsbugのコンソール画面：
```

Wi-Fi connected to "your_ssid"
IP address 192.168.0.17
```
コンソール画面のIPアドレスで
webブラウザーでアクセスすると
以下のように表示される；
```

hello, client at path /.
```

### httpget
ビルド実行：
```
cd $MODDABLE/examples/network/http/httpget

mcconfig -d -m -p esp32/m5atom  ssid=your_ssid password=your_passswd
```

xsbugのコンソール画面：
```

Wi-Fi connected to "your_ssid"
IP address 192.168.0.4
<body>
<div>
    <h1>Example Domain</h1>
    <p>This domain is for use in illustrative examples in documents. You may use this
    domain in literature without prior coordination or asking for permission.</p>
    <p><a href="https://www.iana.org/domains/example">More information...</a></p>
</div>
</body>
</html>
```


### httpgetjson
ビルド実行：
```
cd $MODDABLE/examples/network/http/httpgetjson

mcconfig -d -m -p esp32/m5atom  ssid=your_ssid password=your_passswd
```

xsbugのコンソール画面：
```

Wi-Fi connected to "your_ssid"
IP address 192.168.0.4
The temperature in Menlo Park is 60.21 F.
The weather condition is Clouds.
```

### httpserverbmp
ビルド実行：
```
cd $MODDABLE/examples/network/http/httpserverbmp

mcconfig -d -m -p esp32/m5atom  ssid=your_ssid password=your_passswd
```

xsbugのコンソール画面：
```

Wi-Fi connected to "your_ssid"
IP address 192.168.0.19
```
コンソール画面のIPアドレスで
webブラウザーでアクセスする。
BMP画像が表示される。

### httppost
ビルド実行：
```
cd $MODDABLE/examples/network/http/httppost

mcconfig -d -m -p esp32/m5atom  ssid=your_ssid password=your_passswd
```

xsbugのコンソール画面：
```

Wi-Fi connected to "your_ssid"
IP address 192.168.0.26
name: Moddable
value: 123
```
このサンプルで利用しているテスト用サーバーについては以下を参照のこと：   
[httpbin - HTTP通信のテストに便利なWebサービス＆ソフト](https://www.softantenna.com/wp/review/httpbin/)

### socketreadwrite
ビルド実行：
```
cd $MODDABLE/examples/network/socket/socketreadwrite

mcconfig -d -m -p esp32/m5atom  ssid=your_ssid password=your_passswd
```

xsbugのコンソール画面：
```

Wi-Fi connected to "your_ssid"
IP address 192.168.0.15
socket created.
Socket message connect
Socket message dataSent VALUE = 5744
Socket message dataReceived VALUE = 1436
HTTP/1.1 200 OK
Accept-Ranges: bytes
Age: 501726
Cache-Control: max-age=604800
Content-Type: text/html; charset=UTF-8
Date: Sun, 20 Sep 2020 12:26:21 GMT
Etag: "3147526947"
Expires: Sun, 27 Sep 2020 12:26:21 GMT
Last-Modified: Thu, 17 Oct 2019 07:18:26 GMT
Server: ECS (sjc/4E74)
Vary: Accept-Encoding
X-Cache: HIT
Content-Length: 1256
Connection: close

<!doctype html>
<html>
<head>
    <title>Example Domain</title>

    <meta charset="utf-8" />
    <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style type="text/css">
    body {
        background-color: #f0f0f2;
        margin: 0;
        padding: 0;
        font-family: -apple-system, system-ui, BlinkMacSystemFont, "Segoe UI", "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
        
    }
    div {
        width: 600px;
        margin: 5em auto;
        padding: 2em;
        background-color: #fdfdff;
        border-radius: 0.5em;
        box-shadow: 2px 3px 7px 2px rgba(0,0,0,0.02);
    }
    a:link, a:visited {
        color: #38488f;
        text-decoration: none;
    }
    @media (max-width: 700px) {
        div {
            margin: 0 auto;
            width: auto;
        }
    }
    </style>    
</head>

<body>
<div>
    <h1>Example Domain</h1>
    <p>This domain is for use in illustrative examples in documents. You may use th
Socket message dataReceived VALUE = 190
is
    domain in literature without prior coordination or asking for permission.</p>
    <p><a href="https://www.iana.org/domains/example">More information...</a></p>
</div>
</body>
</html>
```

##  他のサンプル
他のgitのサンプルを動かしてみる。
以下の手順でダウンロードする：
```

mkdir ~/moddable_test
cd ~/moddable
git clone https://github.com/meganetaaan/moddable-examples.git
```
サンプルの簡単な説明は以下を参照のこと：  
https://github.com/meganetaaan/moddable-examples/blob/master/README.md  


### bongo
ボンゴ(打楽器)のデモ：
```
cd ~/moddable_test/moddable-examples/bongo
mcconfig -d -m -p esp32/m5stack_fire 
```

### pomodoro
タイマーデモ：
```

cd ~/moddable_test/moddable-examples/pomodoro
mcconfig -d -m -p esp32/m5stack_fire 
```


### neopixel
内蔵のneopixelを光らせる：
```

cd ~/moddable_test/moddable-examples/unit/neopixel
mcconfig -d -m -p esp32/m5stack_fire 
```
実行すると、内蔵neopixelが色を変えて光る

### light-server
LEDが制御できるhttpサーバー：
```

cd ~/moddable_test/moddable-examples/network/light-server
mcconfig -d -m -p esp32/m5stack_fire ssid=your_ssid password=your_passwd

または
mcconfig -d -m -p esp32/m5atom ssid=your_ssid password=your_passwd

```

xsdebugのコンソール画面：
```

Wi-Fi connected to "your_ssid"
IP address 192.168.0.19
[object Object]probe for m5stack
probe 1
probe 2
probe 3
probe claimed m5stack
```
webブラウザーで以下のurlでアクセスする：
```
http://192.168.0.19/off
The light is off

http://192.168.0.19/on
The light is on

http://m5stack.local/on
The light is on

http://m5stack.local/off
The light is off

```
lightのオン/オフに応じて本体のLEDがオン/オフする。

## シンプルなblink
接続：  
M5Atomに抵抗付きLEDをG25に接続する。(5V,G(ND)も接続する)  

~/moddable_test/blink/main.js
```js

import Timer from 'timer';
import Digital from 'pins/digital';

let led = 25;   // IO25 for M5Atom
let toggle = 0;

// Blink LED
Timer.repeat(() => {
  Digital.write(led, toggle);
  toggle = ~toggle;
}, 100);
```
~/moddable_test/blink/manifest.json
```

{
	"include": [
		"$(MODDABLE)/examples/manifest_base.json",
		"$(MODULES)/pins/digital/manifest.json",
	],
	"modules": {
		"*": "./main",
	},
}
```

ビルド実行：
```

cd ~/moddable_test/blink
mcconfig -m -p esp32/m5atom
```
実行するとG25に接続したLEDが点滅する。


## 参照情報

https://github.com/Moddable-OpenSource/moddable/blob/public/documentation/Moddable%20SDK%20-%20Getting%20Started.md  

[versaアプリ「HR/LCD clock」の紹介](https://beta-notes.way-nifty.com/blog/2019/02/versahrlcd-cloc.html)   

obniz  
https://obniz.com/ja/doc/reference/  

Tessel1   
https://github.com/tessel/t1-docs   
https://github.com/tessel/hardware/blob/master/tessel-hardware-overview.md  
Tessel2  
https://tessel.gitbooks.io/t2-docs/content/  
https://tessel.gitbooks.io/t2-docs/content/Hardware/Tessel_2_Overview.html  

以上
