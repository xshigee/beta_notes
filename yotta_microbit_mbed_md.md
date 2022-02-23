
2021/1/9  
初版  

yotta micro:bit MBED tool
# yotta micro:bit MBED tool

## 概要
micro:bit Yotta開発ツール(linux版)   
なお、本ツールは、microbit-v1用である。

## インストール手順
```bash

mkdir microbit_ws
cd microbit_ws

# 必要なライブラリやツールをインストールする
sudo apt-get update && sudo apt-get install python-setuptools  cmake build-essential ninja-build python-dev libffi-dev libssl-dev && sudo easy_install pip

# yotta　をインストールする
sudo pip install yotta

# micoribt-samples　をダウンロードする
git clone https://github.com/lancaster-university/microbit-samples
cd microbit-samples

# yottaの設定
yt target bbc-microbit-classic-gcc

# src/main.cppがターゲットソースになる
yt build

# ビルド結果(hex)をmicrobitに転送する
# <USER>は実際の環境に合わせる
cp build/bbc-microbit-classic-gcc/source/microbit-samples-combined.hex /media/<USER>/MICROBIT/

```
なお、source/examples　に他のサンプルがある。

## 参考情報

・[Yotta/Installation on Linux](https://lancaster-university.github.io/microbit-docs/offline-toolchains/#yotta)  

・[yotta Documentation](http://docs.yottabuild.org/#introduction)  
・[yotta: Build Software with Reusable Components](https://github.com/ARMmbed/yotta)   

https://os.mbed.com/users/MACRUM/notebook/microbit/  
micro:bit の使い方  

https://github.com/toyowata/microbit_web_bluetooth/blob/master/README.md  
Web Bluetooth（micro:bitとbluetooth接続してブラウザから制御する） 


以上
