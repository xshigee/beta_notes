
2020/2/8

MaixPy Update(v0.5.0_12)
# MaixPy Update(v0.5.0_12)

## 概要
MaixPyのfirmwareのUpdateについて記する。(ubuntu環境でのやり方について述べる)

## 事前準備
(1)ampyのインストール   
AMPY_PORTは、自分の環境に合わせる。
```bash

pip install adafruit-ampy
export AMPY_PORT=/dev/ttyUSB0
export AMPY_BAUD=115200
export AMPY_DELAY=0.5

```
(2)picocomのインストール  
```bash

sudo apt-get install picocom
```
以上のうち、exportしているものは、.bashrcに登録することを勧める。

(3)書き込みツール(kflash)のインストール  
以下のコマンドでkflashをインストールする：
```bash

sudo pip3 install kflash

```

## MaixDuinoのMaixPyのUpdate
```bash

wget http://dl.sipeed.com/MAIX/MaixPy/release/master/maixpy_v0.5.0_12_g284ce83/maixpy_v0.5.0_12_g284ce83_with_lvgl.bin
# 実際にはwgetではエラーになったので、ブラウザーでダウンロードした。
# (エラー原因は不明)

kflash -p /dev/ttyUSB0 maixpy_v0.5.0_12_g284ce83_with_lvgl.bin
# 出力ログ例
[INFO] COM Port Selected Manually:  /dev/ttyUSB0 
[INFO] Default baudrate is 115200 , later it may be changed to the value you set. 
[INFO] Trying to Enter the ISP Mode... 
._
[INFO] Automatically detected goE/kd233 

[INFO] Greeting Message Detected, Start Downloading ISP 
Downloading ISP: |=============================================| 100.0% 10kiB/s
[INFO] Booting From 0x80000000 
[INFO] Wait For 0.1 second for ISP to Boot 
[INFO] Boot to Flashmode Successfully 
[INFO] Selected Flash:  On-Board 
[INFO] Initialization flash Successfully 
Programming BIN: |=============================================| 100.0% 9kiB/s
[INFO] Rebooting... 

```

## M5StickVのMaixPyのUpdate
```bash

wget http://dl.sipeed.com/MAIX/MaixPy/release/master/maixpy_v0.5.0_12_g284ce83/maixpy_v0.5.0_12_g284ce83_m5stickv.bin
# 実際にはwgetではエラーになったので、ブラウザーでダウンロードした。
# (エラー原因は不明)

kflash -p /dev/ttyUSB0 maixpy_v0.5.0_12_g284ce83_m5stickv.bin
# 出力ログ例
[INFO] COM Port Selected Manually:  /dev/ttyUSB0 
[INFO] Default baudrate is 115200 , later it may be changed to the value you set. 
[INFO] Trying to Enter the ISP Mode... 
._
[INFO] Automatically detected goE/kd233 

[INFO] Greeting Message Detected, Start Downloading ISP 
Downloading ISP: |=============================================| 100.0% 10kiB/s
[INFO] Booting From 0x80000000 
[INFO] Wait For 0.1 second for ISP to Boot 
[INFO] Boot to Flashmode Successfully 
[INFO] Selected Flash:  On-Board 
[INFO] Initialization flash Successfully 
Programming BIN: |=============================================| 100.0% 10kiB/s
[INFO] Rebooting... 
```

## 実行確認(M5StickV)

```bash

$ picocom /dev/ttyUSB0 -b115200

MicroPython v0.5.0-12-g284ce83 on 2019-12-31; Sipeed_M1 with kendryte-k210
Type "help()" for more information.
>>> 
>>> import gc
>>> gc.collect()
>>> gc.mem_free()
507584

# 内蔵モジュール表示
>>> help('modules')
KPU               gc                pye_mp            uio
Maix              hashlib           random            ujson
__main__          heapq             re                ulab
_boot             image             sensor            uos
_thread           json              socket            urandom
_webrepl          lcd               struct            ure
array             machine           sys               usocket
audio             math              time              ustruct
binascii          math              ubinascii         utime
board             micropython       ucollections      utimeq
builtins          modules           ucryptolib        uzlib
cmath             nes               uctypes           video
collections       network           uerrno            zlib
errno             os                uhashlib
fpioa_manager     pmu               uheapq
Plus any modules on the filesystem
>>> 

```
原因は、まだ不明だが、ときどき/dev/ttyUSB0が接続していても消える場合がある。ホスト環境に原因があるのかもしれないが
ホストPCを再起動すると修復できるようだ。

以上
 
