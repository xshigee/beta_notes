
2021/2/9+

VS Code extension PicoGo for MicroPython
# VS Code extension PicoGo for MicroPython

## 概要
以下のPicoボードのMicroPythonをVS_CodeのextensionのPico-Goでプログラミングする。   
(ホストPCとしてはubuntuを想定している)

・[Raspberry Pi Pico](https://www.switch-science.com/catalog/6900/)  


## 準備
以下のものをインストールする：
```

sudo apt install nodejs

pip install micropy-cli
pip install pylint

cd ~/Downloads
wget https://github.com/cpwood/Pico-Stub/archive/main.zip
uzip main.zip
cd Pico-Stub-main/stubs
micropy stubs add micropython-rp2-1_13-290
```
VS_Code_extension'Pico-Go'のインストール：   
VS_CodeのなかでCtrl-Pを押して  
以下を実行する：
```

ext install ChrisWood.pico-go
```
参照：
[Pico-Go](https://marketplace.visualstudio.com/items?itemName=ChrisWood.pico-go)  

## プロジェクトの作成
以下の手順でプロジェクトを作成する：
```

mkdir proj
cd proj
micropy init
# メニューが表示されるので、以下の２つを選択して改行を押す
[VScode setting ...]
[Pylint MicroPython setting ..]
# その後、以下を選択する
[rp2-micropython-1.13]
# 以上でプロジェクトの雛形が作成される

# MicroPythonが書き込まれたPicoボードをUSB接続する
# 以下でVS Codeを起動する
code .
```
なお、projは。任意のプロジェクト名とする。

## プログラムの作成
起動しているVS_Codeで
新規ファイルとして
flash.pyを作成して   
以下の内容に編集する：

flash.py
```python

from time import sleep
from machine import Pin

led = Pin(25, Pin.OUT)

while True:
    led.value(1)
    sleep(0.1)
    led.value(0)
    sleep(0.1)
```

## プログラムの実行
VS_Codeの最下行の[Run]をクリックすると
プログラムが実行される。

もう一度、[Run]をクリックすると実行が中止される。（トグル動作）

上のやり方で、上手く実行できなかったときは
最下行の[Pico Connected]をクリックして
いったん接続を切り
最下行の[Pico Disconnected]をクリックして
再度、接続する。

なお、このやり方の実行の場合、RAMでプログラムが実行される。

## フラッシュへの書き込み

Picoボードに書き込む場合は、
最初に実行されるのがboot.pyなので
以下のboot.pyを作成する：

boot.py
```python

print("boot start..")
import flash
```
上のプログラムでは、boot.pyから
flash.pyを呼び出す形で
プログラムが実行される。

最下行の[Upload]をクリックすると
Picoボードにカレントディレクトリにあるファイルが書き込まれる。   
実行例：
```

Failed to read project status, uploading all files
[1/5] Writing file boot.py (0kb)
[2/5] Writing file dev-requirements.txt (0kb)
[3/5] Writing file flash.py (0kb)
[4/5] Writing file micropy.json (0kb
[5/5] Writing file requirements.txt (0kb)
Upload done, resetting board...
OK
```
upload後、プログラムが自動に実行される。  
実行を止めたい場合、TERMINAL画面でCtrl-Cを押す。

これで止まらない場合は、いったん、接続を切ってから
再接続して、Ctrl-Cを押す。

なお、uploadが上手く実行できなかったときは  
以下を実行する：  
(1)最下行メニューから[All commands]/[Pico-Go > Delete all files from boardt]を選択する。  
(2)・確認のダイヤログが画面の最上行に現れるので、[Cancel][Yes]のなかから[Yes]を選択する。   
(3)TERMINAL画面に「All files and directories have been deleted from the board.」が表示され  
   Picoボードのなかのファイルをすべて削除される。  
(4)[Uplaod]をクリックする。  

## 参照情報

・[micropython-pico PDF download](https://hackspace.raspberrypi.org/books/micropython-pico/pdf/download)   

・[Pico-Go VS Code Extension](https://marketplace.visualstudio.com/items?itemName=ChrisWood.pico-go)     
・[Developing for the Raspberry Pi Pico in VS Code — Getting Started](https://medium.com/all-geek-to-me/developing-for-the-raspberry-pi-pico-in-vs-code-getting-started-6dbb3da5ba97)  
・[Developing for the Raspberry Pi Pico in VS Code — Start Coding!](https://medium.com/all-geek-to-me/developing-for-the-raspberry-pi-pico-in-vs-code-start-coding-bb3834233eff)  

以上
