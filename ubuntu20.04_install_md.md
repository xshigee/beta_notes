
2021/2/17  
「開発ツールに関係するインストール」を追加した。  

2020/10/12:  
bluetoothの動かし方を追加した。  

2020/10/3:  
初版  

ubuntu20.04 install
# ubuntu20.04 install

## 概要
ubuntu20.04をインストールする。   
日経Linux(2020/9)のインストールDVDを使用して、新規にPCのHDDをSSDに換装してインストールした際の注意事項などをまとめた。

## 注意事項/留意事項
(1)DVDインストールの際にハードに依存したモジュールなどのインストールをするオプションを使用しないこと。PCのハードウェアに依存すると思うが、再ブート時にエラーになりlinuxが起動しなくなる。  
(2)SNAPベースのアプリのインストールがサポートされたが、実際に使用した印象では完成度が低いので、従来の「sudo apt install xxxx」を使用する。  
例)leafpadはsnapベースだがインストール後、起動するが、エラーが起きて使いものならなかった。VLCは、間違えてインストール・ボタンを２度押したらエラーが出て、インストールできなくなった。  
(3)bluetoothが動作していない。(古いバージョンのutbuntuでは動作していた)  
(4)python3.xに移行したようで、python2.xはデフォルトではインストールされていない。

## 最初にやること(ディクトリの英語化)

ホームディレクトリ内のディレクトリ名を英語表記にするには   
以下を実行する：
```

LANG=C xdg-user-dirs-gtk-update
```

ウィンドウが開くので、" Don't ask me this again " にチェックを入れて、
[ Update Names ] をクリックする。


## 問題のあったアプリのインストール方法
### leafpad
以下、実行する：
```bash
sudo apt install gdebi
wget http://archive.ubuntu.com/ubuntu/pool/universe/l/leafpad/leafpad_0.8.18.1-5_amd64.deb
# debファイルを左クリックしてgdebiインストーラーを起動する
```
参照：「[Ubuntu 20.04にSnap版ではなくdebパッケージ版のLeafpadをインストール
(Snap版のLeafpadで不具合発生)](https://ubuntuapps.net/blog-entry-1121.html)」

### VLC
以下、実行する：
```bash
sudo apt install vlc
sudo apt install vlc-plugin-access-extra libbluray-bdj libdvdcss2
#このインストール後、「libdvd-pkg： `apt-get check`」失敗のようなエラーが出るようになるなら以下を実行する
sudo dpkg-reconfigure libdvd-pkg
```
参照：「[libdvd-pkg： `apt-get check`が失敗しました。パッケージが壊れている可能性があります。](https://qastack.jp/ubuntu/1094062/libdvd-pkg-apt-get-check-failed-you-may-have-broken-packages-aborting)」

### bluetooth
まだ、解決していないが、以下のリンクが参考になるかもしれない(いうまでもなく、動作することは保証しない)  
・[Ubuntu 20.04 bluetooth not working](https://askubuntu.com/questions/1231074/ubuntu-20-04-bluetooth-not-working)  
・[Miix2 8のLubuntu20.04LTS設定方法](https://blog.goediy.com/?p=441#toc1)  

私のハードウェアの場合、以下のようなエラーが出ているようだ：
```
$ dmesg | grep -i bluetooth
[    3.101690] Bluetooth: Core ver 2.22
[    3.101707] Bluetooth: HCI device and connection manager initialized
<省略>
[    3.274510] bluetooth hci1: Direct firmware load for brcm/BCM20702A1-0a5c-21e6.hcd failed with error -2
[    3.274513] Bluetooth: hci1: BCM: Patch brcm/BCM20702A1-0a5c-21e6.hcd not found
<省略>
[   24.820848] Bluetooth: RFCOMM ver 1.11
```

## その他のインストール
以下、必要そうなものを以下のようにインストールした：
```
sudo apt install net-tools
sudo apt install picocom
sudo apt install python3-distutils
sudo apt install python3-venv
sudo apt install curl
sudo apt install git
```

## 開発ツールに関係するインストール
```

sudo apt update

sudo apt install cmake
sudo apt install libusb-dev
sudo apt install libusb-1.0-0-dev

# lib for Arduino IDE for microbit
sudo apt install libudev1:i386

# for Arduino IDE for ESP32/ESP8266
sudo apt install python-is-python3
pip install pyserial
pip uninstall serial
```
参照:  
[Fixing the Arduino IDE for the ESP32/ESP8266 on Ubuntu 20.04](https://koen.vervloesem.eu/blog/fixing-the-arduino-ide-for-the-esp32esp8266-on-ubuntu-2004/)   

## 追記(bluetoothの動かし方)
以下の手順でbluetoothを動かすことができた：
```

sudo apt install blueman
bluetoothctl
# bluetoothctrlが起動したら以下のコマンドを実行する：
# scan on
# pair xxxxx
# (xxxxxは、ペアリングしたいデバイスのアドレス)
# quit
```
以上で希望のデバイスのペアリングができる。  
ubuntuを再起動すると最上行にbluetoothのアイコンが出現するので
それをクリックすことで他のデバイスのペアリングなどができる。  
ペアリング済みのキーボードなどは再起動しても、自動的に接続される。  

推測になるが、私の環境では、内蔵のbluetoothとble用にUSBドングルを刺しているが
内蔵のものがエラーになっていて、USBドングルのものはbluemanで起動できているようだ。  
したがって、標準の設定/bluetoothでは(内臓のもののみに対応しているらしく)
デバイス名を表示しない。  
bluetoohアイコン(blueman)のものは、内蔵とUSBドングルに対応しているので
USBドングルに切り替えて使用する。


## 参考情報

bluetooth関係:
   
https://qiita.com/propella/items/6daf3c56e26f709b4141  
Linux のコマンドラインで Bluetooth 接続   

https://mumeiyamibito.0am.jp/bluetoothctl  
無銘闇人の備忘録/bluetoothctl  

https://askubuntu.com/questions/1231074/ubuntu-20-04-bluetooth-not-working  
Ubuntu 20.04 bluetooth not working  

https://blog.goediy.com/?p=441  
Miix2 8のLubuntu20.04LTS設定方法  

https://qiita.com/maachan_9692/items/256adb8f83e7658d51d5  
ubuntu 20.04 でSONY のヘッドホンWH-1000XM4 をLDAC で接続したい！！  

https://freefielder.jp/blog/2016/07/ubuntu-bluetooth-tethering.html  
Ubuntu / CHIP】Bluetoothテザリング接続をコマンドラインから。  

以上
