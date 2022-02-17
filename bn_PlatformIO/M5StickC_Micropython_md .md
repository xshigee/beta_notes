
2020/2/3+

M5StickC MicroPython Install
# M5StickC MicroPython Install

## 概要
M5StickCにMicroPythonのfirewareを書き込む方法について述べる。
ここでは、linux環境でのインストール方法について説明する。   

## 準備
(1)ampyのインストール   
AMPY_PORTは、自分の環境に合わせる。
```bash


pip install adafruit-ampy
export AMPY_PORT=/dev/ttyUSB0
export AMPY_BAUD=115200
```


(2)udevのruleの設定   
以下を実行するとシリアル(/dev/ttyACMx,/dev/ttyUSBx)が使用可能になる。  
（PlatformIOをインストール済みなら設定完了につき不要）
```bash


curl -fsSL https://raw.githubusercontent.com/platformio/platformio-core/master/scripts/99-platformio-udev.rules | sudo tee /etc/udev/rules.d/99-platformio-udev.rules
sudo service udev restart
sudo usermod -a -G dialout $USER
sudo usermod -a -G plugdev $USER
```

(3)piccomのインストール
```bash

sudo apt-get install piccom
```
以上のうち、exportしているものは、.bashrcに登録することを勧める。

## Firewareのダウンロード＆書き込み
ここでは、他のESP32対応のMicroPythonのファームウェアも存在するが、ここではM5StickC専用(UIFlow対応)のものをM5Burnerで書き込む。

以下のようにする：
```bash

mkdir M5SC
cd M5SC
wget https://m5stack.oss-cn-shenzhen.aliyuncs.com/resource/software/M5Burner_Linux.zip
unzip M5Burner_Linux.zip
# 以上で M5Burner のインストールが終了する

# M5StickCをPCに接続する
# 以下で　M5Burner　が起動する。
./M5Burner

```

```

M5Burnerが起動したら
(1)左側のメニューから以下を選択する。
UIFlo3-v1.4.4
#ダウンロードアイコンが表示されている場合、まだダウンロードされていないので
#ダウンロードアイコンを押してダウンロードを実行する。

(2)以下のように設定する：
COM:/dev/ttyUSB0
Baudrate:750000
Series:StickC

(3)[Burn]を押して書き込みを実行する。

```

## UIFlowのIDEを使用する。
UIFlowはcloudタイプのIDEで、blockyとpythonの両方でプログラミングできる。

(1)WiFi設定   
以下を参考にして、SSID,PASSWDを設定する。   
[M5StickC WiFi Setup](https://docs.m5stack.com/#/ja/quick_start/m5stickc/m5stickc_quick_start_with_uiflow?id=_2-wi-fi%E3%82%BB%E3%83%83%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0)   

(２) UIFlowに接続する   
http://flow.m5stack.com/ にブラウザでアクセスする。    
アクセスしたら最上行の一番左側のプルダウンメニューの中に設定があるので、
設定画面に入り以下を行なう：   
・M5StickCの画面に表示されているApi_Keyを入力する   
・Languageを好きなもの選択する   
・Deviceに「Stick-C」を選択する   

以上を設定したら設定から抜ける。

これでIDEとして使用可能な状態になる:   
・Pythonで使用したい場合は、最上行のメニューの[Blocky/Python]の切り替えから[Python]を選ぶ。   
・ブラウザ画面の最下行にM5StickCとの接続状態の表示があるので「Disconnected」になっているようなら
その右側の同期アイコンをクリックして「Connected」にする。   

## ＬＣＤ/ボタン制御プログラム

以下のプログラムをUIFlowにコピー＆ペーストして実行する。    
HATを持っていないので、そこの部分を削除してLCDとボタンを使用しているのみの
プログラムに変えた。  

M5SC_demo_lcd.py 
```python

# demo without Hat
from m5stack import *
from m5ui import *
from uiflow import *
import hat

#setScreenColor(0x111111)
setScreenColor(0x0)

label0 = M5TextBox(48, 1, " Press btn A Run/Stop", lcd.FONT_Default,0xff0378, rotate=90)

flag = None
i = None

def buttonA_wasPressed():
  global flag, i
  flag = not flag
  label0.setFont(lcd.FONT_Default)
  if flag:
    label0.setText(' BeetlcC Run!')
    hat_BeetleC0.SetPulse(0, 40)
    hat_BeetleC0.SetPulse(1, 40)
  else:
    label0.setText(' BeetlcC Stop!')
    hat_BeetleC0.SetPulse(0, 0)
    hat_BeetleC0.SetPulse(1, 0)
  pass
btnA.wasPressed(buttonA_wasPressed)

```

## WiFi確認プログラム

以下のプログラムをUIFlowにコピー＆ペーストして実行する。   
実行中の出力はシリアル(/dev/ttyUSB0)に出力されるので、実行を開始したら、   
picocom /dev/ttyUSB0 -b115200   
などでM5StickCとＰＣを接続すること。  

以下の部分は自分のWiFi環境に合わせること   
WIFI_SSID = "your_ssid"   
WIFI_PASSWD = "your_passwd"  

M5SC_starwars.py 
```python

# NIC setup
import network, socket

WIFI_SSID = "your_ssid"
WIFI_PASSWD = "your_passwd"

def wlan_connect(ssid='SSID', password='PASSWD'):
    import network
    wlan = network.WLAN(network.STA_IF)
    if not wlan.active() or not wlan.isconnected():
        wlan.active(True)
        print('connecting to:', ssid)
        wlan.connect(ssid, password)
        while not wlan.isconnected():
            pass
  

wlan_connect(ssid=WIFI_SSID, password=WIFI_PASSWD)

#==========================================

# Star Wars ASCII Art (python3)

import socket
addr_info = socket.getaddrinfo("towel.blinkenlights.nl",23)
addr = addr_info[0][-1]

s = socket.socket()
s.connect(addr)

while True:
   data = s.recv(500)
   print(str(data, 'utf8'), end='')

#=============================================


```

## UIFlowを使用しない方法(その１)(/flashに書き込む）

/flash/flow.pyが最初に実行されるプログラムになっているので、この内容を差し替えれば、UIFlow無しでプログラムが実行できる。
なので以下のようにする：
```bash

# UIFlowを使いたい時に戻す必要があるのでバックアップをとる
ampy get flow.py M5SC_flow.py
# xxxx.pyにflow.pyを差し替える 
ampy put xxxx.py /flash/flow.py
# このあと、電源を切り(6秒長押し)　再度、電源を入れる(2秒長押し)
# 自動的にxxxx.pyが起動する

# UIFlowを再度使用する場合、flow.pyを元に戻す
ampy put M5SC_flow.py /flash/flow.py

```

## UIFlowを使用しない方法(その２)(RAM実行）

RAMで実行する場合が以下のようにする：
```bash

ampy run xxxx.py

```

## MicroPythonのバージョン

```python
>>> os.uname()
(sysname='esp32', nodename='esp32', release='1.11.0', version='v1.11-319-g54c3f2613-dirty on 2020-01-15', machine='ESP32 module with ESP32')
>>> 

```

## 起動時の切り替え方法

再起動後、UIFlowクラウドメッセージが表示されたらボタンA[M5]をすばやく押すと、メニューに入ることができる。   
メニューとしては以下の３つが表示される：
```

・Program
・APP.lst
・Setup
```
これらは、ボタンB[正面から見て右側]で選択でき、ボタンA[M5]で確定できる。  
Programでは、main.pyが自動実行される。  
APP.ListではプログラムListが表示され選択したプログラムが実行できる。（らしいが、ちゃんと動作しているかどうか不明）  

Setupでは：  
以下の３つの選択ができる：
```

・Cloud mode
・USB mode
・Wi-Fi Setting
```
・Cloud_modeでは、インターネットに自動的にアクセスに行き、、CloudのIDEと接続する。  
・USB_modeでは、インターネットに自動的にアクセスしなくなり、普通のPythonボードと同様にUSBシリアルで接続できる状態になる。どっちのモードでもシリアル接続可能なので、違いは、ネットワークアクセスに行かない分、ネットワークアクセスで無駄に時間を取らなくなること。  
・WiFi_Settingでは、WiFiの設定ができる。入力しやすさを考えるとM5StickCのAPに接続する方法のほうが簡単なような気がする。  


以上
