
2020/2/13

MicroPython Tool pyboard.py
# MicroPython Tool pyboard.py

## 概要
MicroPythonのツールとしてpyboad.pyを紹介する。本家のdocに説明があるのだが、気が付かなかった。MicroPythonボードをホストＰＣに接続しておくと、シェルからPCのpythonのプログラムを実行するのと、同じ感覚でMicroPythonのプログラムを実行できる。シリアル接続でREPLに入れるボードなら何でも使用可能のようなので、pyboard,ESP32などのボードで使用できる。

## ツールのインストール
以下を実行する：
```bash

wget https://github.com/micropython/micropython/raw/master/tools/pyboard.py
sudo chmod -x pyboard.py

```

## 実行例
```bash

# STM32などシリアルが/dev/ttyACM0になるボードで使用する場合
./pyboard.py --device /dev/ttyACM0 -c 'print(1+1)'

# ESP32などシリアルが/dev/ttyUSB0になるボードで使用する場合
./pyboard.py  --device /dev/ttyUSB0 -c 'print("Hello")'
./pyboard.py  --device /dev/ttyUSB0 ESP32_starwars.py
# startwarsのASCII-Artのアニメがシェル画面上に流れる
...

```

## 参考情報  

[The pyboard.py tool](https://docs.micropython.org/en/latest/reference/pyboard.py.html)  
[ESP32のMicroPythonのインストール方法](https://beta-notes.way-nifty.com/blog/2020/02/post-fd9771.html)  
・このなかに「ESP32_starwars.py」がある  

[ＥＳＰ３２－ＤｅｖＫｉｔＣ　ＥＳＰ－ＷＲＯＯＭ－３２開発ボード](http://akizukidenshi.com/catalog/g/gM-11819/)    
[MicroPython - Quick reference for the ESP32](https://docs.micropython.org/en/latest/esp32/quickref.html#)    
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)　　　


以上
