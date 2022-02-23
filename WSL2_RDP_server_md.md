
2021/3/22  
・日本語変換のインストール方法について追加した。

2021/3/21+   
・「「localhost:3390」で接続する際、接続できないことがある。」問題があり、   
それは、localhostの実IPへの変換が安定していないせいだったが、  
.wslconfig設定を追加することで修正した。

2021/3/18++  
・SSH環境を修正したので、RDPログイン環境下でも、PowerShell.exeが動作するようになった。  
参照：[WSL2でSSHサーバーを動かす](https://beta-notes.way-nifty.com/blog/2021/03/post-7e6528.html)   
　  
2021/3/17  
初版  
　

WSL2 RDP server
# WSL2 RDP server

## 概要
WSL2でRDPサーバーを動かす。   
以下での環境整備が完了している前提でのRDPサーバーのインストール方法について述べる。  
・[WSL2でSSHサーバーを動かす](https://beta-notes.way-nifty.com/blog/2021/03/post-7e6528.html)  

## .wslconfig設定(2021/3/21追加)
localhostの実IPへの変換を正常にさせるために以下を設定する。

windows側のc:\Users\\<ユーザ名>\\.wslconfigを作成して   
以下の内容を設定する：   

.wslconfig
```

localhostForwarding=True
```
以上を設定した後、以下を実行する：  

PowerShell  
```

wsl --shutdown
```
以上の実行でwslをシャットダウン（その後自動で起動される）。  
これは、windows10のPCを起動するたびに実行する必要があるので  
SSHサーバーで自動実行のbatに登録すると便利になる。  

setup-ubuntu-ssh.bat
```

wsl --shutdown

for /F %%i in ('wsl -d Ubuntu-20.04 exec hostname -I') do set ip=%%i
netsh interface portproxy delete v4tov4 listenport=22
netsh interface portproxy add    v4tov4 listenport=22 connectaddress=%ip%

wsl -d Ubuntu-20.04 -u root bash /opt/service_start.sh
```

## RDPサーバーのインストール
以下の手順でインストールする：    
ubuntu20.04/WSL2
```


sudo apt update && sudo apt -y upgrade
sudo apt -y install xfce4
sudo apt install xfce4 xfce4-goodies

sudo apt install xrdp

sudo cp /etc/xrdp/xrdp.ini /etc/xrdp/xrdp.ini.bak
sudo sed -i 's/3389/3390/g' /etc/xrdp/xrdp.ini
sudo sed -i 's/max_bpp=32/#max_bpp=32\nmax_bpp=128/g' /etc/xrdp/xrdp.ini
sudo sed -i 's/xserverbpp=24/#xserverbpp=24\nxserverbpp=128/g' /etc/xrdp/xrdp.ini
echo xfce4-session > ~/.xsession

#startwm.shを編集する
sudo nano /etc/xrdp/startwm.sh
#最後の2行をコメントアウトする。
#そして、末尾に
#startxfce4
#を追加する。
```

## 編集後のstartwm.sh
/etc/xrdp/startwm.sh  
(修正部分のみ)
```
<省略>

##test -x /etc/X11/Xsession && exec /etc/X11/Xsession
##exec /bin/sh /etc/X11/Xsession

# xfce
startxfce4
```

## RDPサーバーの起動
sshでWSL2-ubuntuにログインして
以下を実行する：  
```

sudo service xrdp start
#出力
* Starting Remote Desktop Protocol server
```

RDPサーバーの起動状況を確認したい場合、  
以下のコマンドを実行する：
```

service xrdp status
#出力
 * xrdp-sesman is running
 * xrdp is running
```

ある程度、自動的にxrdpサーバーを起動する場合は  
以下を~/.profileの末尾に追加する：  
.profile　　
```

<省略>

# xrdp
retval=$(service xrdp status >/dev/null 2>&1; echo $?)
if [ ! $retval == "0" ]; then
echo xrdp server start...
sudo service xrdp start
fi

# dbus
retval=$(service dbus status >/dev/null 2>&1; echo $?)
if [ ! $retval == "0" ]; then
echo dbug start...
sudo service dbus start
fi
```

このスクリプトでログイン時に   
serviceの状態をチェックしているので、    
xrdp serverが停止している場合、   
「xrdp server start...」が表示されるので、   
パスワードを入力する。  

## RDPで接続する

windows10の「リモートデスクトップ接続」で   
localhost:3390 に接続する。  
この後、プロンプトが表示されるので、   
セッションをxorgのままにして、   
WSL2のユーザー名とパスワードを入力する。  
ここで、xfce4のディスクトップ画面が表示される。  

現在、以下のような制限があるようだ：  
(1)PATHが変更になっているのか、PowerShell.exeが動かない。  
2021/3/18: SSH環境の改善で動作するようになった。   
参照：[WSL2でSSHサーバーを動かす](https://beta-notes.way-nifty.com/blog/2021/03/post-7e6528.html)     
(2)コマンドラインからGUIアプリを起動した場合は、(Xserverが起動している場合)Xserverによる表示になる。   
以下の場合はRDPによる表示になる：    
・ファイラーでファイルを指定してアプリを選択してクリックするとGUIアプリを起動する。また、アプリ・アイコンをクリックしての起動やアプリケーション・メニューからの起動などができる。   
(3)外部からのアクセスでもRDP接続できるようだが、画面が黒いままで使用不可である。  
[]()  　　 
[]()  
/********** 以下の解決済み(2021/3/21) **********   
トラブルシュート：   
「localhost:3390」で接続する際、接続できないことがある。  
原因不明だが、windows10のPCのIPアドレスがDHCPで払い出されて、  
実質的にIPアドレスが変わったときに接続できない状況になっている印象がある。  
確定的な解決方法は今のところないが、何度も再起動からトライしていると接続できるようだ。  
************************************************/

## 日本語変換のインストール(2021/3/22)
RDPディスクトップのターミナルで以下を実行する：
```bash

# 日本語化
sudo apt -y install language-pack-ja
sudo update-locale LANG=ja_JP.UTF8

# fcitx-mozc(日本語入力）のインストール
sudo apt install -y fcitx-mozc dbus-x11
sudo sh -c "dbus-uuidgen > /var/lib/dbus/machine-id"

# fcitx設定
# profileの編集
sudo nano /etc/profile
#以下を末尾に追加する：
#fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
export DefaultIMModule=fcitx
fcitx-autostart > /dev/null 2>&1
xset -r 49 > /dev/null 2>&1
##

```
以上のインストールが終了したら、WSL2を再起動する。
再度、RDPログインすれば、日本語変換が使用可能になる。

## おまけ(chromeのインストール)
以下の手順でchromeをインストールできる：
```bash

sudo sh -c 'echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list'
sudo wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
sudo apt update
sudo apt install google-chrome-stable
```
デスクトップのプリケーション・メニューから起動できる。


## 参考情報

localhost関連：  
・[WSL2の2つの設定ファイル「.wslconfig」と「wsl.conf」](https://ascii.jp/elem/000/004/044/4044122/)    
・[「WSL 2」最大の課題であった“localhost”問題が解決 ～「Windows 10」Build 18945
すべてのWSLディストロに共通のグローバル設定、カスタムLinuxカーネルの利用にも対応](https://forest.watch.impress.co.jp/docs/news/1198651.html)  
・[WSL2: UbuntuでLocalhostを表示させる方法](https://zenn.dev/masa20210102/articles/001439eb4ad301)  

RDP関連：   
・[Linux on Windows: WSL with Desktop Environment via RDP](https://dev.to/darksmile92/linux-on-windows-wsl-with-desktop-environment-via-rdp-522g)   
・[WSL2+ubuntu20.04: GUI化して使う方法](https://qiita.com/atomyah/items/887a5185ec9a8206c7c4)   

SSH関連：  
・[WSL2のUbuntuにsshできるようにする](https://sunday-morning.app/posts/2020-07-16-wsl2-ubuntu-ssh)  
・[Windows10で特定のポートを開放する](https://support.borndigital.co.jp/hc/ja/articles/360002711593-Windows10%E3%81%A7%E7%89%B9%E5%AE%9A%E3%81%AE%E3%83%9D%E3%83%BC%E3%83%88%E3%82%92%E9%96%8B%E6%94%BE%E3%81%99%E3%82%8B)   
・[WSL2 に外部からアクセスする](https://bayashi.net/diary/2020/1121)  
・[MobaXterm - Enhanced terminal for Windows with X11 server, tabbed SSH client, network tools and much more](https://mobaxterm.mobatek.net/)  

日本語変換関連：  
・[WSL2+Ubuntu 20.04LTSでデスクトップ(xfce4)を表示する。](https://www.e-nekorakuen.net/?p=21815)  
・[Windows10 の WSL(Windows Subsystem for Linux) の Ubuntu18.04 で日本語入力する設定](https://vogel.at.webry.info/201905/article_6.html)  

その他：(未確認)   
・[WSL2+Ubuntu 20.04環境から音声を出力する](https://astherier.com/blog/2020/08/wsl2-ubuntu-sound-setting/)  


以上
