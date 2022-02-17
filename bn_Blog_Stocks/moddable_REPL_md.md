
2020/9/27:  
初版  

Moddable REPL
# Moddable REPL

## 概要
以下のModdableのREPLを動かす。   
本記事は「[Moddableをインストールする](https://beta-notes.way-nifty.com/blog/2020/09/post-e7940b.html)」の続きになる。
Moddableのインストールは「[Moddableをインストールする](https://beta-notes.way-nifty.com/blog/2020/09/post-e7940b.html)」を参照のこと。

[moddable](https://www.moddable.com/)  

## 修正
関係ソースを修正する：
```
cd $MODDABLE/examples/js/repl
leafpad replcore.js
```

replcare.jsの22行目を以下のように修正する：
```
const newline = "\n";
→
const newline = "\r\n\r\n";
```

## manifest.json修正
(base64のモジュールを組み込むために)以下のように修正する：
$MODDABLE/examples/js/repl/manifest.json
```

{
	"include": [
		"$(MODDABLE)/examples/manifest_base.json",
	],
	"modules": {
		"*": [
			  "./main",
			  "./replcore",
            "$(MODULES)/data/base64/*",
		]
	},
	"preload": [
		"repl",
		"replcore",
        "base64",
	],
	"strip": [],
<以下、変更がないので省略>
```

## ビルド実行
以下の手順でビルド実行する：
```

cd $MODDABLE/examples/js/repl
mcconfig -m -p esp32/m5atom
```

## 出力例(REPL例)
```
mcconfig -m -p esp32/m5atom
<省略>
--- idf_monitor on /dev/ttyUSB0 115200 ---
--- Quit: Ctrl+] | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H ---
ets Jun  8 2016 00:22:57

rst:0x1 (POWERON_RESET),boot:0x13 (SPI_FAST_FLASH_BOOT)
configsip: 188777542, SPIWP:0xee
clk_drv:0x00,q_drv:0x00,d_drv:0x00,cs0_drv:0x00,hd_drv:0x00,wp_drv:0x00
mode:DIO, clock div:1
load:0x3fff0018,len:4
load:0x3fff001c,len:768
ho 0 tail 12 room 4
load:0x40078000,len:9756
load:0x40080400,len:5620
entry 0x40080644


Moddable REPL (version 0.0.1)



> var x = 12
undefined

> x
12

> x + 5
17

> x ** 2
144

> eval("x + 3")
15

> const Timer = require("timer")
ReferenceError: get require: undefined variable

> Object.keys(require.cache)
ReferenceError: get require: undefined variable

> const Base64 = require("base64")
ReferenceError: get require: undefined variable

> 
```
上の出力はビルド画面からlog出力されている例だが、
実行後、「picocom /dev/ttyUSB0 -b115200」で出力することもできる。
「[Compiling JavaScript on Embedded Devices](https://blog.moddable.com/blog/eval/)」の説明では
requireが動作するようだが、実際には、現状のバージョンでは、requireが実装されていない？ようでエラーになっている。

$MODDABLE/examples/js/repl/main.jsを以下のように修正して   
「mcconfig -d -m -p esp32/m5atom」(デバッガ付き)でビルド実行すると
base64のモジュールが動作することが確認できるので、manifest.jsonの修正としては正しいようだ。  
$MODDABLE/examples/js/repl/main.js
```js

import REPL from "repl";
import Base64 from "base64";

let console = new REPL;

trace(Base64.decode("aGVsbG8sIHdvcmxk")+"\n");
trace(Base64.encode("hello, world")+"\n");
```
デバッグ・コンソール画面：
```

[object ArrayBuffer]
aGVsbG8sIHdvcmxk
```

## 参照情報

https://github.com/Moddable-OpenSource/moddable/blob/public/documentation/Moddable%20SDK%20-%20Getting%20Started.md  

[Compiling JavaScript on Embedded Devices](https://blog.moddable.com/blog/eval/)   

以上
