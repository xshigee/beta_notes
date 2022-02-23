
2020/1/12

Raspberry Pi Zero_W Micropython Install
# Raspberry Pi Zero_W Micropython Install

## 概要
Raspberry_Pi_Zero-WにMicropythonをインストールする方法について記載する。(他のRaspberry_Piでも同様の方法でインストールできるはず)


## 事前準備
以下をRaspberryPiにインストールする：    
(fullのimageを使用しているのであれば（たぶん）不要)   
```bash

sudo apt-get install git
sudo apt-get install build-essential
sudo apt-get install libffi-dev
```

## 参照URL
[The MicroPython project](https://github.com/micropython/micropython)   
[Installing MicroPython - How To Guide](https://www.raspberrypi.org/forums/viewtopic.php?t=191744)   
[raspberrypi/hats](https://github.com/raspberrypi/hats)   


## インストール手順
```

mkdir mp
cd mp
git clone https://github.com/micropython/micropython.git
cd micropython

cd mpy-cross
make
出力ログ：
<省略>
CC gccollect.c
LINK mpy-cross
   text	   data	    bss	    dec	    hex	filename
 272484	    396	    436	 273316	  42ba4	mpy-cross
pi@raspberrypi:~/mp/micropython/mpy-cross $ 

cd ..
cd ports/unix
make submodules
make
出力ログ：
<省略>
CC ../../lib/timeutils/timeutils.c
LINK micropython
   text	   data	    bss	    dec	    hex	filename
 302953	   4508	   1248	 308709	  4b5e5	micropython
pi@raspberrypi:~/mp/micropython/ports/unix $ 

```

## 動作確認
(1)簡単な確認
```bash

pi@raspberrypi:~/mp/micropython/ports/unix $ ./micropython 
MicroPython v1.12-58-g7ef2f6511 on 2020-01-12; linux version
Use Ctrl-D to exit, Ctrl-E for paste mode
>>> list(5 * x + y for x in range(10) for y in [4, 2, 1])
[4, 2, 1, 9, 7, 6, 14, 12, 11, 19, 17, 16, 24, 22, 21, 29, 27, 26, 34, 32, 31, 39, 37, 36, 44, 42, 41, 49, 47, 46]
>>>Ctrl-D
 
```
(2)本格的な確認
```bash

cd ~/mp/micropython/
cd tests
./run-tests

出力ログ：
<省略>
pass  unix/time.py
733 tests performed (19641 individual testcases)
733 tests passed
49 tests skipped: builtin_help builtin_next_arg2 builtin_range_binop class_delattr_setattr io_buffered_writer memoryview_itemsize namedtuple_asdict sys_getsizeof cmd_parsetree repl_words_move framebuf1 framebuf16 framebuf2 framebuf4 framebuf8 framebuf_subclass ucryptolib_aes128_ctr urandom_extra urandom_extra_float ure_debug ure_groups ure_span ure_sub ure_sub_unmatched vfs_basic vfs_blockdev vfs_fat_fileio1 vfs_fat_fileio2 vfs_fat_finaliser vfs_fat_more vfs_fat_oldproto vfs_fat_ramdisk vfs_fat_ramdisklarge vfs_lfs vfs_lfs_corrupt vfs_lfs_error vfs_lfs_file vfs_lfs_mount vfs_userfs math_factorial_intbig module_getattr mpy_invalid mpy_native resource_stream schedule sys_settrace_features sys_settrace_generator sys_settrace_loop extra_coverage
pi@raspberrypi:~/mp/micropython/tests $ 

```

## Micropythonの最終インストール
```bash

sudo ln -s ~/mp/micropython/ports/unix/micropython /usr/local/bin/micropython
```
ここで、コマンドとして「micropython」が使用できるようになる。

## micropython-socketライブラリーのインストール
```bash

micropython -m upip install micropython-socket
出力ログ：
Installing to: /home/pi/.micropython/lib/
Warning: micropython.org SSL certificate is not validated
Installing micropython-socket 0.5.2 from https://micropython.org/pi/socket/socket-0.5.2.tar.gz

ライブラリーのインストール確認：
$ micropython
MicroPython v1.12-58-g7ef2f6511 on 2020-01-12; linux version
Use Ctrl-D to exit, Ctrl-E for paste mode
>>> import usocket
>>> print(dir(usocket))
['__class__', '__name__', 'AF_INET', 'AF_INET6', 'AF_UNIX', 'MSG_DONTROUTE', 'MSG_DONTWAIT', 'SOCK_DGRAM', 'SOCK_RAW', 'SOCK_STREAM', 'SOL_SOCKET', 'SO_BROADCAST', 'SO_ERROR', 'SO_KEEPALIVE', 'SO_LINGER', 'SO_REUSEADDR', 'getaddrinfo', 'inet_ntop', 'inet_pton', 'sockaddr', 'socket']
>>> import socket
>>> print(dir(socket))
['__class__', '__name__', '__file__', 'AF_INET', 'AF_INET6', 'AF_UNIX', 'MSG_DONTROUTE', 'MSG_DONTWAIT', 'SOCK_DGRAM', 'SOCK_RAW', 'SOCK_STREAM', 'SOL_SOCKET', 'SO_BROADCAST', 'SO_ERROR', 'SO_KEEPALIVE', 'SO_LINGER', 'SO_REUSEADDR', 'getaddrinfo', 'inet_ntop', 'inet_pton', 'sockaddr', 'socket', '_socket', '_GLOBAL_DEFAULT_TIMEOUT', 'IPPROTO_IP', 'IP_ADD_MEMBERSHIP', 'IP_DROP_MEMBERSHIP', 'INADDR_ANY', 'error', '_resolve_addr', 'inet_aton', 'create_connection']
>>> Ctrl-D

```

## USB serial接続
Raspberry_Pi_Zero-WのUSBにOTGケーブルを接続して（USB serial可能な）Arduinoボードなどを接続すれば、Arduinoのセンサーなどの情報をシリアル経由で取り出せるので、そのデータをmicropythonで処理できる。この方法によって、特別なHATなどをRaspberry_Piに接続せずにセンサーデータを利用できる。（serial portは、/dev/ttyACM0などになる）

以上
