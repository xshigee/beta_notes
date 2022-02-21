
2020/5/13+

PlatformIO Sipeed RV(RISC-V) Debugger Longan Nano
# PlatformIO Sipeed RV(RISC-V) Debugger Longan Nano

## 概要
開発ツールPlatformIOで以下のSipeed-RV(RISC-V)-Debuggerを使用する。ここでは、ターゲット・ボードとして、Longan-Nanoを使用する。
(ホストPCとしてはubuntuを想定している)

[Sipeed RISC-V デバッガ](https://www.switch-science.com/catalog/5712/)  
[Sipeed USB-JTAG/TTL RISC-V Debugger](https://www.seeedstudio.com/Sipeed-USB-JTAG-TTL-RISC-V-Debugger-p-2910.html)  


## PlatformIOのインストール
```bash

python3 -m venv pio_env
source pio_env/bin/activate

pip3 install platformio

インストール後も、本ツールを使用する場合
同じディレクトリで以下を実行する：  
source pio_env/bin/activate
# 「source」は、「.」でも良い

```
## 準備
以下を実行して、platformio用のudevのrulesを登録する：
```bash


curl -fsSL https://raw.githubusercontent.com/platformio/platformio-core/master/scripts/99-platformio-udev.rules | sudo tee /etc/udev/rules.d/99-platformio-udev.rules

sudo udevadm control --reload-rules

sudo usermod -a -G dialout $USER
sudo usermod -a -G plugdev $USER

```

## 接続
以下の表のようにターゲット・ボードLongan-Nanoと本デバッガを接続する：

| pin#  |  Board JTAG Pin |
| ---: | :---: |
| 1 | GND |
| 10 | TCK  |
| 8 | TDO |
| 6 | TMS |
| 4 | RST |
| 2 | TDI |

Longan-Nanoと接続する際は、コネクタ・ピンとしてRST(Reset)が出ていないので
接続しなくてもかまわない。ただし、ボードをリセットするときは、電源をいったん切る必要がある。
(デバッガーを接続しているときは、Resetボタンは無効になっている)

USBシリアルの機能もあるので、USBシリアルとして以下のように接続する：

| pin# | Board Serial Pin|
| ---: | :---: |
|　9 | GND |
| 7 | NC |
| 5  | RXD(←TXD)|
| 3  | TXD(→RXD) |
| 1  | GND(JTAGのGNDとしても使用する) |

NC: None Connection

ホストＰＣに本デバッガ(ドングル)を挿すと,/dev/ttyUSB0,/dev/ttyUSB1 と２つのシリアルポートとして認識されるが
/dev/ttyUSB1が実際のシリアルポートになる。


## サンプル・プログラム

```bash

$ cd pio_ws
$ . pio_env/bin/activate

git clone https://github.com/sipeed/Longan_GD32VF_examples.git
cd Longan_GD32VF_examples

#　32GB以下のmicroSDに以下の２つのファイルをコピーして、
# そのmicroSDをボードのTFスロットに刺しておく。
$ ls put_into_tf_card/
bmp.bin  logo.bin

cd Longan_GD32VF_examples/gd32v_lcd

nano platformio.ini
#以下の内容になるよう編集する：
```
platformio.ini
```
;   Upload options: custom upload port, speed and extra flags
;   Library options: dependencies, extra library storages
;   Advanced options: extra scripting
;
; Please visit documentation for the other options and examples
; https://docs.platformio.org/page/projectconf.html

[env:sipeed-longan-nano]
platform = gd32v
board = sipeed-longan-nano
framework = gd32vf103-sdk
#upload_protocol = dfu
#debug_tool = sipeed-rv-debugger

upload_protocol = sipeed-rv-debugger
debug_tool = sipeed-rv-debugger

#upload_protocol = um232h
#debug_tool = um232h
```
続き：
```bash

pio run -t upload   
#出力例：
$ pio run -t upload
Processing sipeed-longan-nano (platform: gd32v; board: sipeed-longan-nano; framework: gd32vf103-sdk)
----------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/gd32v/sipeed-longan-nano.html
PLATFORM: GigaDevice GD32V 1.1.2 > Sipeed Longan Nano
HARDWARE: GD32VF103CBT6 108MHz, 32KB RAM, 128KB Flash
DEBUG: Current (sipeed-rv-debugger) External (altera-usb-blaster, gd-link, jlink, rv-link, sipeed-rv-debugger, um232h)
PACKAGES: 
 - framework-gd32vf103-sdk 1.0.0 
 - tool-gd32vflash 0.1.0 
 - tool-openocd-gd32v 0.1.1 
 - toolchain-gd32v 9.2.0
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 0 compatible libraries
Scanning dependencies...
No dependencies
Building in release mode
Checking size .pio/build/sipeed-longan-nano/firmware.elf
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
RAM:   [=====     ]  47.9% (used 15696 bytes from 32768 bytes)
Flash: [==        ]  15.7% (used 20632 bytes from 131072 bytes)
Configuring upload protocol...
AVAILABLE: altera-usb-blaster, gd-link, jlink, rv-link, serial, sipeed-rv-debugger, um232h
CURRENT: upload_protocol = sipeed-rv-debugger
Uploading .pio/build/sipeed-longan-nano/firmware.elf
Open On-Chip Debugger 0.10.0+dev-00911-gcfbca74bd (2019-09-12-09:31)
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
Warn : Transport "jtag" was already selected
jtag
adapter speed: 1000 kHz

Info : clock speed 1000 kHz
Info : JTAG tap: riscv.cpu tap/device found: 0x1000563d (mfg: 0x31e (Andes Technology Corporation), part: 0x0005, ver: 0x1)
Warn : JTAG tap: riscv.cpu       UNEXPECTED: 0x1000563d (mfg: 0x31e (Andes Technology Corporation), part: 0x0005, ver: 0x1)
Error: JTAG tap: riscv.cpu  expected 1 of 1: 0x1e200a6d (mfg: 0x536 (Nuclei System Technology Co.,Ltd.), part: 0xe200, ver: 0x1)
Info : JTAG tap: auto0.tap tap/device found: 0x790007a3 (mfg: 0x3d1 (GigaDevice Semiconductor (Beijing)), part: 0x9000, ver: 0x7)
Error: Trying to use configured scan chain anyway...
Warn : AUTO auto0.tap - use "jtag newtap auto0 tap -irlen 5 -expected-id 0x790007a3"
Warn : Bypassing JTAG setup events due to errors
Info : datacount=4 progbufsize=2
Info : Exposing additional CSR 3040
<省略>
Info : Exposing additional CSR 3071
Info : Examined RISC-V core; found 1 harts
Info :  hart 0: XLEN=32, misa=0x40901105
Info : Listening on port 3333 for gdb connections
Info : device id = 0x19060410
Info : flash_size_in_kb = 0x00000080
Info : flash size = 128kbytes
Info : JTAG tap: riscv.cpu tap/device found: 0x1000563d (mfg: 0x31e (Andes Technology Corporation), part: 0x0005, ver: 0x1)
Warn : JTAG tap: riscv.cpu       UNEXPECTED: 0x1000563d (mfg: 0x31e (Andes Technology Corporation), part: 0x0005, ver: 0x1)
Error: JTAG tap: riscv.cpu  expected 1 of 1: 0x1e200a6d (mfg: 0x536 (Nuclei System Technology Co.,Ltd.), part: 0xe200, ver: 0x1)
Info : JTAG tap: auto0.tap tap/device found: 0x790007a3 (mfg: 0x3d1 (GigaDevice Semiconductor (Beijing)), part: 0x9000, ver: 0x7)
Error: Trying to use configured scan chain anyway...
Warn : Bypassing JTAG setup events due to errors
** Programming Started **
** Programming Finished **
** Verify Started **
** Verified OK **
Info : Hart 0 unexpectedly reset!

*** [upload] Error 1
===== [FAILED] Took 6.07 seconds =====

# エラーになっているが「** Verified OK **」となっているので書き込みは正常終了している。
# resetボタンは無効になっているので、いったんUSBケーブルを外してから、再度、接続してボードを再起動する。
# 以上で書き込みが終わり、LCDにデモのアニメーションが表示される。

```

デバッガとして使用する際は、VScode(+platformio)でplatformio.iniを上の例のように設定して、ソース上の行番号の左側をクリックしてbreakpointを設定して、デバッガを起動(Run/Start Debugging)すると、デバッガとして使用できる。
VScodeは以下のコマンドで起動できる：
```bash


code ./project_dir
```

## シリアル・モニタ
プログラム中にprintfなどでシリアル出力しているときは、別の端末で以下のコマンドを実行する：
```bash


picocom /dev/ttyUSB1 -b115200
#または
pio device monitor --baud 115200

```
なお、ビットレート115200は、プログラムの設定で変わっている可能性がある。

## 補足

本デバッガの仕様が以下になっているので、動作確認はしていないが
stlinkとして使用できる可能性がある。
```
仕様

STM32 SWDデバッグインタフェース
シンプルな4線式インタフェース（電源を含む）
STM8 SWIMダウンロードデバッグ
IAR、STVDなどの一般的な開発環境をサポート
ST-LINK Utility 2.0および4.2.1
...
```
接続は以下のようになると思われる：

| pin#  |  Board SWD Pin |
| :--- | :---: |
| 1(GND) | GND |
| 10(TCK) | SWCLK  |
| 6(TMS) | SWDIO |
| 4(RST) | RESET |


参考：
SWD使用時のNucleoのCN4とCPUピンの対応表
| pin#  |  Fuction | CPU port|
| :--- | :---: | :---: |
| 1 | VDD_TARGET | 3.3V |
| 2 | SWCLK  | PA14 |
| 3 | GND | GND |
| 4 | SWDIO | PA13 |
| 5 | NRST |　RESET |
| 6 | SWO | NC |

## 参考情報

[PIO Unified Debugger » Sipeed RV Debugger](https://docs.platformio.org/en/latest/plus/debug-tools/sipeed-rv-debugger.html)   
[PIO Unified Debugger » ST-LINK](https://docs.platformio.org/en/latest/plus/debug-tools/stlink.html)  

[PIO Unified Debugger » UM232H](https://docs.platformio.org/en/latest/plus/debug-tools/um232h.html)  
[開発ツールPlatformIOでFT232Hボードをデバッガーとして使用する(Longan-Nano版)](https://beta-notes.way-nifty.com/blog/2020/05/post-5c077f.html)


[Ｓｉｐｅｅｄ　Ｌｏｎｇａｎ　Ｎａｎｏ　ＲＩＳＣ－Ｖ　ＧＤ３２ＶＦ１０３ＣＢＴ６開発ボード](http://akizukidenshi.com/catalog/g/gK-14678/)   
■主な仕様  
・CPU：GD32VF103CBT6  
・メモリ：128KB Flash/32KB SRAM  
・160×80ドット、0.96インチのフルカラーIPS液晶  
・TFスロット(microSDスロット)  

[Longan Nano PINOUT](https://longan.sipeed.com/assets/longan_nano_pinout_v1.1.0_w5676_h4000_large.png)    
[download Longan Nano Datasheet](https://cn.dl.sipeed.com/LONGAN/Nano/Spec/Sipeed%20longan%20nano%20Datasheet%20V1.0.pdf)


[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

[Longan nano で Hello World!](https://qiita.com/myston/items/d7366021f75dce000c3b)   
[Sipeed Longan Nanoで文字を表示してみる](http://kyoro205.blog.fc2.com/blog-entry-667.html)   
・図形描画の関数の説明がある  
[Sipeed Longan Nano 用の、FONTX2対応LCD描画ライブラリとサンプルプログラム](https://github.com/takamame0205/LonganNanoLCD)  

以上
