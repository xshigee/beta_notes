
2020/5/2

PlatformIO um232h Longan Nano
# PlatformIO um232h Longan Nano

## 概要
開発ツールPlatformIOで以下のFT232Hボードをデバッガーとして使用する。ここでは、ターゲット・ボードとして、Longan-Nanoを使用する。
(ホストPCとしてはubuntuを想定している)

[ＦＴ２３２Ｈ使用ＵＳＢ⇔ＧＰＩＯ＋ＳＰＩ＋Ｉ２Ｃ変換モジュール](http://akizukidenshi.com/catalog/g/gM-08942/)  

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

以下を実行して、FT232H用のudevのrulesを登録する：
```bash

sudo apt-get install libusb-1.0

sudo nano /etc/udev/rules.d/11-ftdi.rules
#以下の内容を作成する：
# /etc/udev/rules.d/11-ftdi.rules
SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="6001", GROUP="plugdev", MODE="0666"
SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="6011", GROUP="plugdev", MODE="0666"
SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="6010", GROUP="plugdev", MODE="0666"
SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="6014", GROUP="plugdev", MODE="0666"
SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="6015", GROUP="plugdev", MODE="0666"

sudo udevadm control --reload-rules

sudo usermod -a -G plugdev $USER
```


## 接続
以下の表のようにターゲット・ボードLongan-NanoとFT232Hボードを接続する：

| FT232H Pin | Longan-Nano JTAG Pin | Description |
| :---: | :---: | :--- |
| GND |	GND | Digital ground |
| D0 |	JTCK | JTAG Return Test Clock |
| D1 |	JTDI | Test Data In |
| D2 | JTDO | Test Data Out |
| D3 |	JTMS | Test Mode State |

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

;PlatformIO Project Configuration File
;
;   Build options: build flags, source filter
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
upload_protocol = um232h
debug_tool = um232h

```
続き：
```bash

pio run -t upload   
#出力例：
```
```
$ pio run -t upload
Processing sipeed-longan-nano (platform: gd32v; board: sipeed-longan-nano; framework: gd32vf103-sdk)
--------------------------------------------------------------------------------------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/gd32v/sipeed-longan-nano.html
PLATFORM: GigaDevice GD32V 1.1.2 > Sipeed Longan Nano
HARDWARE: GD32VF103CBT6 108MHz, 32KB RAM, 128KB Flash
DEBUG: Current (um232h) External (altera-usb-blaster, gd-link, jlink, rv-link, sipeed-rv-debugger, um232h)
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
RAM:   [=         ]   7.1% (used 2328 bytes from 32768 bytes)
Flash: [=         ]  12.3% (used 16160 bytes from 131072 bytes)
Configuring upload protocol...
AVAILABLE: altera-usb-blaster, gd-link, jlink, rv-link, serial, sipeed-rv-debugger, um232h
CURRENT: upload_protocol = um232h
Uploading .pio/build/sipeed-longan-nano/firmware.elf
Open On-Chip Debugger 0.10.0+dev-00911-gcfbca74bd (2019-09-12-09:31)
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
jtag
adapter speed: 8000 kHz

Info : clock speed 8000 kHz
Info : JTAG tap: riscv.cpu tap/device found: 0x1000563d (mfg: 0x31e (Andes Technology Corporation), part: 0x0005, ver: 0x1)
Warn : JTAG tap: riscv.cpu       UNEXPECTED: 0x1000563d (mfg: 0x31e (Andes Technology Corporation), part: 0x0005, ver: 0x1)
Error: JTAG tap: riscv.cpu  expected 1 of 1: 0x1e200a6d (mfg: 0x536 (Nuclei System Technology Co.,Ltd.), part: 0xe200, ver: 0x1)
Info : JTAG tap: auto0.tap tap/device found: 0x790007a3 (mfg: 0x3d1 (GigaDevice Semiconductor (Beijing)), part: 0x9000, ver: 0x7)
Error: Trying to use configured scan chain anyway...
Warn : AUTO auto0.tap - use "jtag newtap auto0 tap -irlen 5 -expected-id 0x790007a3"
Warn : Bypassing JTAG setup events due to errors
Info : datacount=4 progbufsize=2
Info : Exposing additional CSR 3040
Info : Exposing additional CSR 3041
...
<省略>
...
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
===== [FAILED] Took 2.91 seconds ====

# エラーになっているが「** Verified OK **」となっているので書き込みは正常終了している。
# resetボタンは無効になっているので、いったんUSBケーブルを外してから、再度、接続してボードを再起動する。
# 以上で書き込みが終わり、LCDにデモのアニメーションが表示される。

```

デバッガーとして使用する際は、VScode(+platformio)でplatformio.iniを上の例のように設定して、ソース上の行番号の左側をクリックしてbreakpointを設定して、デバッガーを起動(Run/Start Debuggin)すると、デバッガーとして使用できる。

## 参考情報


[PIO Unified Debugger » UM232H](https://docs.platformio.org/en/latest/plus/debug-tools/um232h.html)  


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

