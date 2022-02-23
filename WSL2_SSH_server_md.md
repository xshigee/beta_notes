
2021/3/20  
・Documents/setup-ubuntu-ssh.batの自動起動方法を  
PowerShellの自動起動スクリプトから  
タスクスケジュールによる起動に変更した。  
なので、windows10のPCを起動すると  
自動的にsetup-ubuntu-ssh.batが起動する。  

2021/3/18  
・SSHログイン環境下でも、PowerShell.exeが動作するようにした。

2021/3/15  
・SSHで外部からアクセスした際のDISPLAY変数の設定値を追加した。  
・GUIアプリ起動時に出ているエラーの解消方法について追加した。   

2021/3/13+   
初版  
　

WSL2 SSH server
# WSL2 SSH server

## 概要
WSL2でSSHサーバーを動かす。   
WSL2ならびにbuntu20.04のインストールが終了している前提で
SSHサーバーを動かす方法について述べる。

## SSHサーバーのインストール
以下の手順でインストールする：    
ubuntu20.04/WSL2
```

sudo apt update
sudo apt install openssh-server
```

#  WSL2の起動用スクリプト
WSL2は起動時にデーモンが動いていないので   
以下のサービス起動用のスクリプトを作成する：  
/opt/service_start.sh
```

#!/bin/bash
service ssh start
service dbus start
service avahi-daemon start
service cron start
```
以下のコマンドで実行権限を与える：
```

sudo chmod a+x /opt/service_start.sh
```

## windows10側(PowerShell側)のスクリプト
以下のWSL2側の設定を行うbatを作成する：

Documents/setup-ubuntu-ssh.bat
```
wsl -d Ubuntu-20.04 -u root bash /opt/service_start.sh

for /F %%i in ('wsl -d Ubuntu-20.04 exec hostname -I') do set ip=%%i
netsh interface portproxy delete v4tov4 listenport=22
netsh interface portproxy add    v4tov4 listenport=22 connectaddress=%ip%
```

/********* 以下、起動方法の変更により廃止　*********

該当のbatを自動実行させるために、   
自動実行されるPowerShellのスクリプト(Microsoft.PowerShell_profile.ps1)の  
末尾に以下を追加する：   
Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1   
(存在しない場合、新規に作成する)
```

Documents/setup-ubuntu-ssh.bat
```
これで、PowerShellを管理者権限で立ち上げると   
該当のbatが実行されて
WSL2でSSHサーバーが起動することになる。

*********************************************************/


## タスク・スケジューラによる自動起動の設定

タスク・スケジューラで以下のように設定する：

```

全般：
ユーザーがログオンしているのみ実行する：on
最上位の特権で実行する: on
構成：[windows10] (重要：これを設定しないと動作しないようだ)

トリガー：
タスクの開始: ログオン時
遅延時間を指定する: 30秒間

操作」
操作:[プログラムの開始]
プログラム/スクリプト: Documents/setup-ubuntu-ssh.batを置いたフル・パス

条件：
タスクを実行するためにスリープを解除する：on
それ以外、off
```
参照：[Windows10 - 管理者権限が必要なアプリを自動起動（スタートアップ）](https://pc-karuma.net/windows-10-task-schedule-without-uac-prompt/)   


この設定で、windows10のPCを再起動すると自動的に   
setup-ubuntu-ssh.batが実行される。

これで、<WSL2のユーザー名>@<windowsの外部から見えるIP>で
SSHログインできる。

なお、<windowsの外部から見えるIP>は、
PowerShellで以下を実行した出力から分かる：   
PowerShell
```

ipconfig
# 出力

Windows IP 構成

<省略>

Wireless LAN adapter Wi-Fi:

   接続固有の DNS サフィックス . . . . .:
   リンクローカル IPv6 アドレス. . . . .: xxxxxxxxxxxxxxx
   IPv4 アドレス . . . . . . . . . . . .: 192.168.0.13
   サブネット マスク . . . . . . . . . .: 255.255.255.0
   デフォルト ゲートウェイ . . . . . . .: 192.168.0.1

イーサネット アダプター vEthernet (Default Switch):

＜省略＞

イーサネット アダプター vEthernet (WSL):

＜省略＞
```
WiFiで接続している場合、「Wireless LAN adapter Wi-Fi:」のIPアドレスが
該当のIPになるので、ここでは「192.168.0.13」となる。

## SSHポートを開放する
外部のPCからWSL2のSSHサーバーにアクセスする場合、windows10のSSHポートを開放する必要がある。色々な解説が、ネット上、見つかるが、代表的なものとして以下を挙げる：  
・[Windows10で特定のポートを開放する](https://support.borndigital.co.jp/hc/ja/articles/360002711593-Windows10%E3%81%A7%E7%89%B9%E5%AE%9A%E3%81%AE%E3%83%9D%E3%83%BC%E3%83%88%E3%82%92%E9%96%8B%E6%94%BE%E3%81%99%E3%82%8B)   
または、管理者権限でPowerShellで以下を実行する：   
PowerShell
```

New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Inbound -LocalPort 22 -Action Allow -Protocol TCP
```

## GUIアプリ
XフォーワードでGUIアプリを動かすときに必要となるDISPLAY変数は以下を設定する。
```

export DISPLAY=<windowsの外部から見えるIP>:0.0

#上の例では以下のようになる：
export DISPLAY=192.168.0.13:0.0
```
この設定をWSL2の~/.profileと~/.bashrcの末尾に登録すると  
自動化できる。  
なお、  
~/.profileは、ログイン・シェル用の起動スクリプト、  
~/.bashrcは、インタラクティブ・シェル用のスクリプトになる。  

Xサーバーを内蔵している以下のMobaXtermを使用すれば特別にXサーバーを用意しなくてもWSL2のGUIアプリが使用できる。

SSHで外部からアクセスする場合、   
「ssh -Y user@hostname.local」などで起動すれば、   
GUIアプリも動作するが、DISPLAY変数を以下に設定する必要がある：
```

export DISPLAY=localhost:10.0
```
なお、内部のWSL2のログインでも上の設定でGUIアプリが動くようだが
遅いようなので、内部の使用では、もとの設定のほうが良いようだ。

・[MobaXterm - Enhanced terminal for Windows with X11 server, tabbed SSH client, network tools and much more](https://mobaxterm.mobatek.net/)  
設定は、SSHはデフォルトで、   
WSL2ログインの場合は「Run method: Native connector」に  
する必要がある。

## VScode in WSL2
WSL2にVScodeをインストールして実行すると以下のようなメッセージが出る：
```

To use Visual Studio Code with the Windows Subsystem for Linux, please install Visual Studio Code in Windows and uninstall the Linux version in WSL. You can then use the `code` command in a WSL terminal just as you would in a normal command prompt.
Do you want to continue anyway? [y/N] y
To no longer see this prompt, start Visual Studio Code with the environment variable DONT_PROMPT_WSL_INSTALL defined.
```
ようするに、WSL2ではlinux版ではなくてwindows版を使えと言ってくる。
たぶん以下のような機能があるのでwindows版で十分ということらしい。   

・[VSCodeでWSL2のLinux上のファイルをリモート操作する](https://cartman0.hatenablog.com/entry/2020/07/18/VSCode%E3%81%A7WSL2%E3%81%AELinux%E4%B8%8A%E3%81%AE%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%82%92%E3%83%AA%E3%83%A2%E3%83%BC%E3%83%88%E6%93%8D%E4%BD%9C%E3%81%99%E3%82%8B)  

とは言え、普通の使い方がしたい場合、linux版のほうが良いと感じると思うので
そのまま、このメッセージを無視しても構わない。このメッセージが頻繁にでて鬱陶しいので以下を実行すると出なくなる：
```

export DONT_PROMPT_WSL_INSTALL=1
```
永続的に有効にする場合は、~/.profile,~/.bashrcに登録する必要がある。

参考：VScodeのインストール方法
```

wget http://deb.rug.nl/ppa/mirror/packages.microsoft.com/repos/vscode/pool/main/c/code/code_1.54.2-1615424848_amd64.deb
sudo dpkg -i  code_1.54.2-1615424848_amd64.deb
```

## エラー解消
GUIアプリを起動する際、動作に実害はないが、エラーメッセージが出ているので、その解消方法について述べる：  

(1)「G_IS_DBUS_CONNECTION」
```

GLib-GIO-CRITICAL **: 21:22:13.063: g_dbus_proxy_new: assertion 'G_IS_DBUS_CONNECTION (connection)' failed
```
上のようなエラーが出ている場合、以下を実行すると解消する：
```

sudo apt purge fcitx-module-dbus
```
参考：[Lubuntu - G_Is_Dbus_Connection](https://unix.stackexchange.com/questions/490871/lubuntu-g-is-dbus-connection)  

(2)「icon text-x-generic」
```

Could not load theme icon text-x-generic: ...
```
上のようなエラーが出ている場合、以下を実行すると解消する：
```

sudo apt install gnome-icon-theme
```

## PowerShell.exeを動作させる(2021/3/18)
SSHログイン環境ではPATHの問題か「PowerShell.exe」が動作しない。
そこで以下のようにシンボリック・リンクを張り、SSHログイン環境下でも
「PowerShell.exe」が動作するようにする。  

bash
```bash

sudo ln -s /mnt/c/WINDOWS/System32/WindowsPowerShell/v1.0//PowerShell.exe /usr/bin/PowerShell.exe

#確認
which PowerShell.exe
#出力
/usr/bin/PowerShell.exe
#シンボリック・リンクの確認
ls -l /usr/bin/PowerShell.exe 
lrwxrwxrwx 1 root root 62  3月 18 18:02 /usr/bin/PowerShell.exe -> /mnt/c/WINDOWS/System32/WindowsPowerShell/v1.0//PowerShell.exe
```

## 参考情報

・[WSL2のUbuntuにsshできるようにする](https://sunday-morning.app/posts/2020-07-16-wsl2-ubuntu-ssh)  
・[Windows10で特定のポートを開放する](https://support.borndigital.co.jp/hc/ja/articles/360002711593-Windows10%E3%81%A7%E7%89%B9%E5%AE%9A%E3%81%AE%E3%83%9D%E3%83%BC%E3%83%88%E3%82%92%E9%96%8B%E6%94%BE%E3%81%99%E3%82%8B)   
・[WSL2 に外部からアクセスする](https://bayashi.net/diary/2020/1121)  
・[MobaXterm - Enhanced terminal for Windows with X11 server, tabbed SSH client, network tools and much more](https://mobaxterm.mobatek.net/)  
 
以上
