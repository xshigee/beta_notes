
2020/1/15+

microbit Micropython CLI(Command Line Interface) tool
# microbit MicroPython CLI tool

## 概要
microbitのMicroPythonのプログラミングの書き込み/実行をコマンドラインで行なう方法について説明する。
MicroPythonのfirmwareのhexとMicroPythonのスクリプトの２つをhexとして結合したものを
microbitに書き込んでいるので、firmwareとスクリプトを結合するツールを利用する。

## ツールのインストール
```bash

mkdir microbit_MP
cd microbit_MP
git clone https://github.com/bbcmicrobit/micropython.git

cd micropython/tools
#このディレクトリにあるmakecombinedhex.pyをツールとして使うことになる。
```

## firmwareのダウンロード(その1)
micro:bitの最新版の場合、v2.0のeditorの出力したfirmwareを利用する。
以下のオンラインツールを利用してfirmwareを作る。  
(1)以下のリンクをクリックしてエディタを起動する。   
[microbit-python-editor v2.0](https://python.microbit.org/v/2.0)  
(2)編集して空のスクリプトを作る。  
(3)[Download]を押して、生成したhexをダウンロードする。  
(4)ダウンロードしたhexをfirmwareV20.hexにrenameする。  
(5)このhexファイルを~/microbit_MP/micropython/toolsに置く。  

## firmwareのダウンロード(その2)
互換ボードのchibi:bitの場合、メモリマップの違いか、microbitの最新版用のhexを書き込んだ場合、エラーになり動作しないことがあるので、初期バージョンのpython-editor-mbf0.1.zipに含まれるfirmware.hexをそのまま使用する。  
このhexファイルを~/microbit_MP/micropython/toolsに置く。  
現在はネットに置いていない？ようなので、取り出したfirmware.hexを[ここに](https://beta-notes.way-nifty.com/blog/files/firmware.hex)置く。  
(microbitのみしか使用しないときは、本件は不要)


## 使用例
以下の実行例でmicrobitに書き込むouthexがDownloadのディレクトリにでき、hello.py、compass.pyが入力するスクリプトのファイルになる。入力するスクリプトは任意のエディタで作る。  
```bash書き込み

~/microbit_MP/micropython/tools$ 

./makecombinedhex.py firmwareV20.hex hello.py -o ~/Downloads/out.hex
./makecombinedhex.py firmwareV20.hex ../examples/compass.py -o ~/Downloads/out.hex
```

## 書き込み/実行
上で作ったout.hexを接続しているmicrobitのストレージに書き込む。
書き込んだ後、該当のストレージが消えるが、再度、出現したら書き込み完了になるので
ボードの[reset]を押すと書き込んだプログラムが起動する。


## bashスクリプト定義
上のままだと使いにくいので、以下のようなbashスクリプトを定義する。：   
(chmd u+x filename で実行権限を与えること)

chibibit用：   
(chibibitのストレージなどは自分の環境に合わせること)  
mpy2chibi
```bash

#!/bin/bash
HX=~/microbit_MP/micropython/tools/firmware.hex
PY=$PWD/$1
OUT=~/Downloads/$1_V10.hex
#echo $HX
#echo $PY
#echo $OUT
~/microbit_MP/micropython/tools/makecombinedhex.py $HX $PY -o $OUT
cp $OUT /media/user/CHIBIBIT/
rm $OUT
```

microibt用：  
(microbitのストレージなどは自分の環境に合わせること)  
mpy2mb
```bash

#!/bin/bash
HX=~/microbit_MP/micropython/tools/firmwareV20.hex
PY=$PWD/$1
OUT=~/Downloads/$1_V20.hex
#echo $HX
#echo $PY
#echo $OUT
~/microbit_MP/micropython/tools/makecombinedhex.py $HX $PY -o $OUT
cp $OUT /media/user/MICROBIT/
rm $OUT

```

これらのスクリプトを利用して   
以下のようにで一気に書き込みが行える。

```bash

./mpy2chibi hello.py
./mpy2mb hello.py

```
これで、好みのエディタで作成したスクリプトをmicrobitで実行できる。


## おまけ(offline editor)
以下のやり方でofflineのeditorが使えるようになる。

```bash

git clone https://github.com/bbcmicrobit/PythonEditor
cd PythonEditor
git submodule update --init --recursive

# 以下のコマンドでeditorを起動することができる。。
./bin/show
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...

# http://localhost:8000/editor.htmlをブラウザーでアクセスするとオンラインと同じeditorが起動する。

# 以下のコマンドでもeditorを起動する
firefox editor.html 
# または
chromium-browser editor.html 

```

参考参照:   
[Python Editor Version 2](https://support.microbit.org/support/solutions/articles/19000102970-python-editor-version-2)  
version 2.0になってから、WebUSB(chromeのみ)を利用して、cmsis-dap経由で、microbitにプログラムを書き込んだり、シリアルでデータをやり取りできるようになり、便利になった。   

以上
