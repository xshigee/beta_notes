
2020/9/24+:   
別のconsole.logを追加した。

2020/9/24:  
初版  

Moddable console.log
# Moddable console.log

## 概要
以下のModdableにconsole.logを追加する。   
本記事は「[Moddableをインストールする](https://beta-notes.way-nifty.com/blog/2020/09/post-e7940b.html)」の続きになる。
Moddableのインストールは[Moddableをインストールする](https://beta-notes.way-nifty.com/blog/2020/09/post-e7940b.html)を参照のこと。

[moddable](https://www.moddable.com/)  

## console.logを追加する
[Moddable SDKを使ってJavaScriptでIoT開発してみた](https://dotstud.io/blog/developed-iot-using-moddable/)のconsole.logの追加方法を利用したが、そのままでは、動作しなかったので、manifest.json,main.jsを変更した。

### manifest.json
proj/manifest.json
```

{
	"include": "$(MODDABLE)/examples/manifest_base.json",
	"modules": {
		"*": [
			"./main",
            "./esp/*",
		]
	}
}
```

### main.js
proj/main.js
```javascript

import Timer from "timer";
//import console from "console";
import "console";

const a = 'a';
const z = 'z';

let c = a;
let i = 0;

Timer.repeat(() => {
  console.log(`${String(Date.now()).padStart(15)}:${i}`+'\r');
  i = (i >= 10) ? 0 : i + 1;
}, 1000);

Timer.repeat(() => {
  console.log(`${String(Date.now()).padStart(15)}:${c}`+'\r');
  c = (c >= z) ? a : String.fromCharCode(c.charCodeAt(0) + 1);
}, 1500);
```

### console.c
proj/esp/console.c
```c

#include "xsAll.h"
#include "xs.h"

void xs_console_destructor(void)
{
}

void xs_console_log(xsMachine *the)
{
  int argc = xsToInteger(xsArgc), i;

  for (i = 0; i < argc; i++) {
    char *str = xsToString(xsArg(i));

    do {
      uint8_t c = c_read8(str);
      if (!c) {
        ESP_putc('\n');
        break;
      }

      ESP_putc(c);

      str++;
    } while (1);
  }
}
```

### console.js
proj/esp/console.js
```javascript

class Console @ "xs_console_destructor" {
  static log() @ "xs_console_log"
  static debug() {
    trace(args);
  }

  constructor() {
  }
}
Object.freeze(Console.prototype);

global.console = Console;
```

## ビルド実行
以下の手順で実行する：
```
M5Atomの場合：
mcconfig -m -p esp32/m5atom

M5Stackの場合：
mcconfig -m -p esp32/m5stack
```
デバッガを起動したい場合、「-d」を追加する。
ただし、console.log出力しているので
デバッグのコンソール画面には、何も表示されない。

## 出力例
```
mcconfig -m -p esp32/m5atom
<省略>

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
           1023:0
           1523:a
           2023:1
           3023:2
           3023:b
           4023:3
           4523:c
           5023:4
           6023:5
           6024:d
           7023:6
           7523:e
           8023:7
           9023:8
           9024:f
          10023:9
          10523:g
          11023:10
          12023:0
          12024:h
          13023:1
          13523:i
          14023:2
          15023:3
          15023:j
...
```
上の出力はビルド画面からlog出力されている例だが、
実行後、「picocom /dev/ttyUSB0 -b115200」で出力することができる。

## 別のconsole.log
moddable-polyfillにもconsole.logがあるので、そのダウンロードについて説明する。
以下の手順でダウンロードする：
```

mkdir polyfill
cd polyfill
git clone https://github.com/horihiro/moddable-polyfill.git
cd moddable-polyfill/example/timer
```

### ビルド実行
以下の手順で実行する：
```
M5Atomの場合：
mcconfig -d -m -p esp32/m5atom  ssid=your_ssid password=your_passwd

M5Stackの場合：
mcconfig -d -m -p esp32/m5stack ssid=your_ssid password=your_passwd
```
breakpointが設定されて、デバッグ画面で停止するので[▶]を押して再スタートする。
ただし、console.log出力しているのでデバッグのコンソール画面には、何も表示されない。

### 出力例
```
<省略>
Detecting chip type... ESP32
Chip is ESP32-PICO-D4 (revision 1)
Features: WiFi, BT, Dual Core, 240MHz, Embedded Flash, VRef calibration in efuse, Coding Scheme None
Crystal is 40MHz
MAC: 50:02:91:91:19:20
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 1500000
Changed.
Configuring flash size...
Compressed 18352 bytes to 11964...
Wrote 18352 bytes (11964 compressed) at 0x00001000 in 0.1 seconds (effective 1114.9 kbit/s)...
Hash of data verified.
Compressed 3072 bytes to 144...
Wrote 3072 bytes (144 compressed) at 0x00008000 in 0.0 seconds (effective 6153.2 kbit/s)...
Hash of data verified.
Compressed 783520 bytes to 478987...
Wrote 783520 bytes (478987 compressed) at 0x00010000 in 7.6 seconds (effective 825.6 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...
<省略>

Done
#以下、console.log出力

fired
1600919068256

1600919069256

1600919070256

1600919071256

```
main.jsからbreakpointを外し(debuggerを外し)
[-d]を外してビルドし直すと、「picocom /dev/ttyUSB0 -b115200」でconsole.log出力を表示できる。


### manifest.json
manifest.json
```

{
  "config": {
    "sntp": "pool.ntp.org",
  },
  "include": [
    "../../esp/console/manifest_console.json",
    "../../all/timer/manifest_timer.json",
  ],
  "modules": {
    "*": [
      "./main",
    ],
  },
}
```

### main.js
main.js
```javascript

debugger;

import {} from 'w3c_timer';
import {} from 'console';

const id_timeout = setTimeout(() => {
  console.log('fired');
}, 2000);

const id_interval = setInterval(() => {
  console.log(Date.now());
}, 1000);
```
polyfillでsetTimeout/setIntervalも実装されている。


## 参照情報

https://github.com/Moddable-OpenSource/moddable/blob/public/documentation/Moddable%20SDK%20-%20Getting%20Started.md  

[moddable 用 Polyfill ライブラリーを作ってます](https://uncaughtexception.hatenablog.com/entry/2019/03/18/181733)   

以上
