
2021/1/5  
初版  

ubuntu Live USB
# ubuntu Live USB

## 概要
ubuntu_Live_USBの作成方法を記載する   
(ホストはubuntuを想定している)

## 準備
真っさらなUSBストレージ(16GBくらい)か
FAT32で初期化したUSBストレージを用意する。

## 手順
```

# Live USB に使用するISOフォーマットのディストリビューションをダウンロードする
mkdir iso
cd iso

wget http://cdimage.ubuntulinux.jp/releases/20.04.1/ubuntu-ja-20.04.1-desktop-amd64.iso

# unetbootin　をインストールする
sudo add-apt-repository ppa:gezakovacs/ppa
sudo apt update
sudo apt install unetbootin

# 準備したUSBをホストPCに刺す
# (間違って書き込まないために、他のUSBデバイスは外す)

# 実行する
sudo QT_X11_NO_MITSHM=1 /usr/bin/unetbootin

# (1)起動したらラジオ・ボタンが２つあるので、
#  ・「デストリビューション(ネットワークからダウンロード)」か
#  ・「ディスクイメージ(ローカルのISOファイルを指定)」をボタンで選ぶ。
# (2)「スペースは...(Ubuntuのみ)」のところは1024MBぐらいの値を入れる。
# (3)タイプ(USBドライブ)、ドライブ(/dev/sdxx)　が、現在、刺しているUSBを示しているか確認する
# (4)[OK]を押す

```
実行(書き込み)が完了するとUSB内のブートイメージができあがる。

注意：  
ディストリビューションは日本語版を選ぶこと!    
そうでないと起動した際にキーボードレイアウトが   
日本仕様でなくなるので記号入力が困難になる。  


## 参考情報

・[UNetbootin : Create bootable Live USB drives for Windows and MacOS](https://unetbootin.org/)  
・[UbuntuでUbuntuのLive USBの作成する](https://jitaku.work/it/os/linux/ubuntu/live-usb/)   

以上
