
2020/1/9+++

headless RaspberryPi
# headless RaspberryPi

## 概要
表示装置を接続しないでＰＣを運用することをheadlessというが、ここでは、最初のbootからRaspberryPiをheadlessで使う方法についてまとめた。基本的には本家siteのドキュメントにあるとおりだが、それに必要なものを追加した。ここでは、 Raspberry_Pi_Zero-Wのボードを使う前提で説明してある。


## 事前準備(clientPC側(linux))
以下のdeamonをインストールする：
```bash

sudo apt-get update
sudo apt-get install avahi-daemon
```
sshのポートを開くための設定：
```

sudo ufw allow ssh
```

## 参照URL
[installation/installing-images/linux](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md)      
[configuration/wireless/headless](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md)   
[configuration/wireless/wireless-cli](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md)   
[remote-access/ssh/unix](https://www.raspberrypi.org/documentation/remote-access/ssh/unix.md)  
[How to open ssh port using ufw on Ubuntu/Debian Linux](https://www.cyberciti.biz/faq/ufw-allow-incoming-ssh-connections-from-a-specific-ip-address-subnet-on-ubuntu-debian/)   

## Downloading raspbian image
以下から希望のimageをダウンロードする。   
[raspbian_full_latest](https://downloads.raspberrypi.org/raspbian_full_latest)   
[raspbian_latest](https://downloads.raspberrypi.org/raspbian_latest)   
[raspbian_lite_latest](https://downloads.raspberrypi.org/raspbian_lite_latest)   

## Discovering the SD card mountpoint and unmounting it
imageに書き込むSDのデバイス名の確認し、アンマウントする。
```
lsblk -p

<省略>
...
/dev/mmcblk0     179:0    0  28.9G  0 disk 
└─/dev/mmcblk0p1 179:1    0  28.9G  0 part /media/user/18F0-D045
```

これで書き込むSDのデバイス名が/dev/mmcblk0と確認できた。
これをunmaunt(取り出す)する。(物理的には取り出さない)

## Downloadしたimageを書き込む

実行例：
(/dev/mmcblk0は自分の環境に合わせて変更する)
```
cd <downloadしたimageがあるディレクトリ>

unzip -p 2019-09-26-raspbian-buster.zip | sudo dd of=/dev/mmcblk0 bs=4M conv=fsync
または
unzip -p 2019-09-26-raspbian-buster-lite.zip | sudo dd of=/dev/mmcblk0 bs=4M conv=fsync
または
unzip -p 2019-09-26-raspbian-buster-full.zip | sudo dd of=/dev/mmcblk0 bs=4M conv=fsync

```

出力ログ例:
```
$ unzip -p 2019-09-26-raspbian-buster.zip | sudo dd of=/dev/mmcblk0 bs=4M conv=fsync
0+40350 レコード入力
0+40350 レコード出力
3829399552 bytes (3.8 GB, 3.6 GiB) copied, 268.143 s, 14.3 MB/s

または

$ unzip -p 2019-09-26-raspbian-buster-full.zip | sudo dd of=/dev/mmcblk0 bs=4M conv=fsync
[sudo] komatsu のパスワード: 
0+62011 レコード入力
0+62011 レコード出力
6811549696 bytes (6.8 GB, 6.3 GiB) copied, 470.142 s, 14.5 MB/s 

または

$ unzip -p 2019-09-26-raspbian-buster-lite.zip | sudo dd of=/dev/mmcblk0 bs=4M conv=fsync
0+26539 レコード入力
0+26539 レコード出力
2248146944 bytes (2.2 GB, 2.1 GiB) copied, 158.167 s, 14.2 MB/s

```

## bootディレクトリのファイル追加

書き込んだSDをいったん取り外し、再度、挿入する。
すると、bootディレクトリが出現する。

(1)ssh有効化   
bootにsshという名前のファイルを作成する。(ファイル内容は空で良い)

(2)wireless設定   
bootにwpa_supplicant.confという名前で
以下の内容のファイルを作成する：

wpa_supplicant.conf  
(複数のSSIDを設定する例)
```

ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=JP

network={
    ssid="<Name of your WiFi>"
    psk="<Password for your WiFi>"
    priority=1
    id_str="<ID1（任意)>"
}


network={
    ssid="<Name of your WiFi>"
    psk="<Password for your WiFi>"
    priority=2
    id_str="<ID2（任意)>"
}
```
接続するssidの方式はWAPかWPA2であること  
(WEPでは接続できないようだ)   

(3)該当SDを取り出す
boot,rootfsをアンマウントする。

## Boot
書き込んだSDをRaspberryPiのボードに刺し、電源をオンする。(起動する)  
ボードのLEDが点滅し始めて、常灯するまで待つ。

## Login
ホストPCで接続を確認するために   
(名前解決に時間がかかるので)  
以下のコマンドを成功するまで繰り返す。   
ping raspberrypi.local  

出力ログ例：
```

$ ping raspberrypi.local
PING raspberrypi.local (192.168.0.26) 56(84) bytes of data.
64 bytes from 192.168.0.26: icmp_seq=1 ttl=64 time=39.4 ms
64 bytes from 192.168.0.26: icmp_seq=2 ttl=64 time=78.2 ms
64 bytes from 192.168.0.26: icmp_seq=3 ttl=64 time=10.0 ms
64 bytes from 192.168.0.26: icmp_seq=4 ttl=64 time=9.81 ms
64 bytes from 192.168.0.26: icmp_seq=5 ttl=64 time=18.9 ms
^C
--- raspberrypi.local ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4005ms
rtt min/avg/max/mdev = 9.816/31.306/78.254/25.833 ms
```
どうしても接続確認ができないようなら「arp -a」を入力して、接続中のPCのIPアドレスを表示させて、RaspberryPiのIPアドレスを推定する。   

接続確認が終わったら以下のコマンドを入力してloginする：   
(デフォルトのパスワードはraspberry)
```

ssh pi@raspberrypi.local
または
ssh -Y pi@raspberrypi.local
(接続後にグラフィクス・アプリを使用する場合)
```
ログイン初期接続のログ出力例:   
```
ssh pi@raspberrypi.local
The authenticity of host 'raspberrypi.local (192.168.0.26)' can't be established.
ECDSA key fingerprint is SHA256:v3pvQPmIvpInAIIOkzFU6FMqri9k4oTWiFI0LNiJHgw.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'raspberrypi.local,192.168.0.26' (ECDSA) to the list of known hosts.
pi@raspberrypi.local's password: 
Linux raspberrypi 4.19.75+ #1270 Tue Sep 24 18:38:54 BST 2019 armv6l

The programs included with the Debian GNU/Linux system are free software;

the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Sep 26 01:32:10 2019

SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.
```

## login後
ログイン中（ログ出力例）:   
```bash

pi@raspberrypi:~ $ uname -a
Linux raspberrypi 4.19.75+ #1270 Tue Sep 24 18:38:54 BST 2019 armv6l GNU/Linux

pi@raspberrypi:~ $ df
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/root       29458568 3118176  25092456  12% /
devtmpfs          217208       0    217208   0% /dev
tmpfs             221536       0    221536   0% /dev/shm
tmpfs             221536    3188    218348   2% /run
tmpfs               5120       4      5116   1% /run/lock
tmpfs             221536       0    221536   0% /sys/fs/cgroup
/dev/mmcblk0p1    258095   53034    205061  21% /boot
tmpfs              44304       0     44304   0% /run/user/1000

pi@raspberrypi:~ $ ls
Desktop    Downloads  Music     Public     Videos
Documents  MagPi      Pictures  Templates

pi@raspberrypi:~ $ ifconfig
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.26  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 fe80::aab:d0bc:bf6e:3058  prefixlen 64  scopeid 0x20<link>
        ether b8:27:eb:bc:01:7c  txqueuelen 1000  (Ethernet)
        RX packets 12575  bytes 14102468 (13.4 MiB)
        RX errors 0  dropped 2  overruns 0  frame 0
        TX packets 4932  bytes 508646 (496.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

pi@raspberrypi:~ $ sudo apt-get update
Get:1 http://archive.raspberrypi.org/debian buster InRelease [25.2 kB]
Get:2 http://raspbian.raspberrypi.org/raspbian buster InRelease [15.0 kB]
Get:3 http://raspbian.raspberrypi.org/raspbian buster/main armhf Packages [13.0 MB]
Get:4 http://archive.raspberrypi.org/debian buster/main armhf Packages [260 kB]
Fetched 13.3 MB in 2min 4s (107 kB/s) 
Reading package lists... Done
```

## hostname変更
必要があれば、hostnameを変更するために   
以下の２つのファイルに記述されているhostnameを新しいものにエディタで変更(編集)し   
再起動する：   
(boot前にSDのrootfsのものを変更しても良い)  
```bash

sudo nano /etc/hostname
sudo nano /etc/hosts
sudo reboot

```

## グラフィクス・アプリ起動例
```bash
ssh -Y pi@raspberrypi.local

geany &

sudo apt-get install scratch
scratch &

sudo apt-get install leafpad
leafpad &

```

## chromeのインストール
```bash
sudo apt-get install chromium-browser
chromium-browser &

```

## node.jsとrubyのインストール
```
sudo apt-get install nodejs
sudo apt-get install ruby

```

## C#のインストール
```bash

sudo apt-get install mono-complete
mcs --version
Mono C# compiler version 5.18.0.240
csharp
Mono C# Shell, type "help;" for help

Enter statements below.
csharp> Environment.OSVersion
Unix 4.19.75.0
csharp>
Ctrl-D
```

## WiringPiのインストール例
```bash

git clone https://github.com/WiringPi/WiringPi
cd WiringPi
./build
cd ~

gpio -v
gpio version: 2.60
Copyright (c) 2012-2018 Gordon Henderson
This is free software with ABSOLUTELY NO WARRANTY.
For details type: gpio -warranty

Raspberry Pi Details:
  Type: Pi Zero-W, Revision: 01, Memory: 512MB, Maker: Sony 
  * Device tree is enabled.
  *--> Raspberry Pi Zero W Rev 1.1
  * This Raspberry Pi supports user-level GPIO access.
pi@raspberrypi:~ $ gpio readall
 +-----+-----+---------+------+---+-Pi ZeroW-+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 |     |     |    3.3v |      |   |  1 || 2  |   |      | 5v      |     |     |
 |   2 |   8 |   SDA.1 |   IN | 1 |  3 || 4  |   |      | 5v      |     |     |
 |   3 |   9 |   SCL.1 |   IN | 1 |  5 || 6  |   |      | 0v      |     |     |
 |   4 |   7 | GPIO. 7 |   IN | 1 |  7 || 8  | 0 | IN   | TxD     | 15  | 14  |
 |     |     |      0v |      |   |  9 || 10 | 1 | IN   | RxD     | 16  | 15  |
 |  17 |   0 | GPIO. 0 |   IN | 0 | 11 || 12 | 0 | IN   | GPIO. 1 | 1   | 18  |
 |  27 |   2 | GPIO. 2 |   IN | 0 | 13 || 14 |   |      | 0v      |     |     |
 |  22 |   3 | GPIO. 3 |   IN | 0 | 15 || 16 | 0 | IN   | GPIO. 4 | 4   | 23  |
 |     |     |    3.3v |      |   | 17 || 18 | 0 | IN   | GPIO. 5 | 5   | 24  |
 |  10 |  12 |    MOSI |   IN | 0 | 19 || 20 |   |      | 0v      |     |     |
 |   9 |  13 |    MISO |   IN | 0 | 21 || 22 | 0 | IN   | GPIO. 6 | 6   | 25  |
 |  11 |  14 |    SCLK |   IN | 0 | 23 || 24 | 1 | IN   | CE0     | 10  | 8   |
 |     |     |      0v |      |   | 25 || 26 | 1 | IN   | CE1     | 11  | 7   |
 |   0 |  30 |   SDA.0 |   IN | 1 | 27 || 28 | 1 | IN   | SCL.0   | 31  | 1   |
 |   5 |  21 | GPIO.21 |   IN | 1 | 29 || 30 |   |      | 0v      |     |     |
 |   6 |  22 | GPIO.22 |   IN | 1 | 31 || 32 | 0 | IN   | GPIO.26 | 26  | 12  |
 |  13 |  23 | GPIO.23 |   IN | 0 | 33 || 34 |   |      | 0v      |     |     |
 |  19 |  24 | GPIO.24 |   IN | 0 | 35 || 36 | 0 | IN   | GPIO.27 | 27  | 16  |
 |  26 |  25 | GPIO.25 |   IN | 0 | 37 || 38 | 0 | IN   | GPIO.28 | 28  | 20  |
 |     |     |      0v |      |   | 39 || 40 | 0 | IN   | GPIO.29 | 29  | 21  |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+-Pi ZeroW-+---+------+---------+-----+-----+
pi@raspberrypi:~ $ 

```

 ## bluetoothのインストール例
```bash

sudo apt-get install -y pi-bluetooth
sudo reboot

bluetoothctl
Agent registered
[bluetooth]# scan on
Discovery started
#出力例
[CHG] Controller B8:27:EB:43:FE:83 Discovering: yes
[NEW] Device 57:80:9B:6B:4C:EC 57-80-9B-6B-4C-EC
[NEW] Device 5F:55:F2:80:A7:2B 5F-55-F2-80-A7-2B
[NEW] Device CE:EA:67:62:B6:7B Bryton Cadence
[CHG] Device 57:80:9B:6B:4C:EC RSSI: -67
[CHG] Device 57:80:9B:6B:4C:EC RSSI: -81
Ctrl-D
[bluetooth]# quit
pi@raspberrypi:~ $ 

```

## WiFi無しRaspberryPiのHeadless化
WiFi無しボード「Raspberry Pi Model B」でも、今回の方法で作成したSDを刺し、有線LANに接続して起動すると、そのままheadlessとして使用できる。無線の設定であるwpa_supplicant.confを、そのまま残したままでも特に問題はないようだ。
WiringPiをインストールして「gpio readall」を実行したログ出力は以下のようになる：   
(Pinのアサイン状況が分かる)   
```

gpio readall
 +-----+-----+---------+------+---+-Model B1-+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 |     |     |    3.3v |      |   |  1 || 2  |   |      | 5v      |     |     |
 |   2 |   8 |   SDA.1 |   IN | 1 |  3 || 4  |   |      | 5v      |     |     |
 |   3 |   9 |   SCL.1 |   IN | 1 |  5 || 6  |   |      | 0v      |     |     |
 |   4 |   7 | GPIO. 7 |   IN | 1 |  7 || 8  | 1 | ALT0 | TxD     | 15  | 14  |
 |     |     |      0v |      |   |  9 || 10 | 1 | ALT0 | RxD     | 16  | 15  |
 |  17 |   0 | GPIO. 0 |   IN | 0 | 11 || 12 | 0 | IN   | GPIO. 1 | 1   | 18  |
 |  27 |   2 | GPIO. 2 |   IN | 0 | 13 || 14 |   |      | 0v      |     |     |
 |  22 |   3 | GPIO. 3 |   IN | 0 | 15 || 16 | 0 | IN   | GPIO. 4 | 4   | 23  |
 |     |     |    3.3v |      |   | 17 || 18 | 0 | IN   | GPIO. 5 | 5   | 24  |
 |  10 |  12 |    MOSI |   IN | 0 | 19 || 20 |   |      | 0v      |     |     |
 |   9 |  13 |    MISO |   IN | 0 | 21 || 22 | 0 | IN   | GPIO. 6 | 6   | 25  |
 |  11 |  14 |    SCLK |   IN | 0 | 23 || 24 | 1 | IN   | CE0     | 10  | 8   |
 |     |     |      0v |      |   | 25 || 26 | 1 | IN   | CE1     | 11  | 7   |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+-Model B1-+---+------+---------+-----+-----+
```

## WiFi-USBドングル付きRaspberryPiのHeadless化
「Raspberry Pi 2 Model B」にWiFi-USBドングルを付けたものでも、今回の方法で作成したSDを刺し、起動させると、そのままWiFi接続可能なheadlessとして使用できる。    
(該当のWiFi-USBドングルはlinuxで動作実績のあるもの)   
なお、実験した感じでは、「Raspberry Pi 2 Model B」しか、このやり方では起動しないようだ。(USBドライバーの相違が原因？)   
以下、ログ出力例(RAM容量、ボード種類などの確認):
```

$ free -h
              total        used        free      shared  buff/cache   available
Mem:          926Mi       118Mi       315Mi        18Mi       492Mi       733Mi
Swap:          49Mi          0B        49Mi

$ cat /proc/cpuinfo
processor	: 0
model name	: ARMv7 Processor rev 5 (v7l)
BogoMIPS	: 38.40
Features	: half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32 lpae evtstrm 
CPU implementer	: 0x41
CPU architecture: 7
CPU variant	: 0x0
CPU part	: 0xc07
CPU revision	: 5
<省略>
Hardware	: BCM2835
Revision	: a21041
Serial		: 000000007d5c9e71
Model		: Raspberry Pi 2 Model B Rev 1.1

$ uname -a
Linux raspberrypi 4.19.75-v7+ #1270 SMP Tue Sep 24 18:45:11 BST 2019 armv7l GNU/Linux

```

## RaspberryPi4のHeadless化
「Raspberry Pi 4 Model B」に、今回の方法で作成したSDを刺し、起動させることができる。    
以下、ログ出力例(RAM容量、ボード種類の確認):
```

$ free -h
              total        used        free      shared  buff/cache   available
Mem:          3.8Gi        99Mi       3.3Gi        16Mi       441Mi       3.6Gi
Swap:          49Mi          0B        49Mi

$ cat /proc/cpuinfo
processor	: 0
model name	: ARMv7 Processor rev 3 (v7l)
BogoMIPS	: 108.00
Features	: half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32 lpae evtstrm crc32 
CPU implementer	: 0x41
CPU architecture: 7
CPU variant	: 0x0
CPU part	: 0xd08
CPU revision	: 3
<省略>
Hardware	: BCM2835
Revision	: c03112
Serial		: 10000000f8cd9a91
Model		: Raspberry Pi 4 Model B Rev 1.2

$ uname -a
Linux raspberrypi 4.19.75-v7l+ #1270 SMP Tue Sep 24 18:51:41 BST 2019 armv7l GNU/Linux
```

## Bug?
fullイメージのSDで起動した場合、「Raspberry Pi 4 Model B」では、/home/piには以下の状態になっていた。
```bash

$ ls
MagPi
```

他のボードでは、以下のようなディレクトリができる：
```bash

$ ls
Desktop  Documents  MagPi  Pictures  Templates
Downloads  Music  Public    Videos

```

たぶん、「Raspberry Pi 4 Model B」では、初ブート時にディレクトリを作成するスクリプトがエラーになっていると推測する。
空のディレクトリを作っているだけなので、必要があれば、シェルのコマンドでディレクトリを作れば良いだけだと思う。

## WiFi接続性が悪い問題
sshにログインして使った印象では、ＷｉＦｉ接続性の良い順に並べると、WiFiドングル付き、RaspberryPiZero_W、ＲａｓｐｂｅｒｒｙＰｉ４(以下、「4」とする)になり、「4」が一番悪い。「4」の表示回路のノイズやCPUのノイズがWiFi電波に被ってWiFiの接続性能が落ちていると推測している。(RaspberryPiZero_WのWiFi接続性は特に問題ないレベルであった)   
ネット情報にも以下のような記事がありRapberryPiのオンボードのWiFiは性能が悪いことは事実のようだ。　　

[Raspberry Piで無線LANの反応が悪い時の対処法](http://denshikousaku.net/fix-sluggish-response-of-raspberry-pi-wifi-adaptor)   
[Raspberry Pi 4の「特定の解像度でWi-Fiが不安定になる」問題を実際に検証してみた](https://gigazine.net/news/20191206-raspberry-pi-4-wifi-problem/)  

気のせいかもしれないが、SD作成後の初ブート時のWiFi接続性と、それ以降の起動のWiFi接続性を比較すると、初ブート時のほうが良好だった。（ノイズ源となるべき回路の動作状況が異なっている？？？）  
   
自分の環境でも、以下のように/etc/rc.localにHDMIのオフを入れたが「４」は改善しなかった。  
(headless前提なのでHDMIをオフしても問題ない)   
/etc/rc.local
```bash
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

# Print the IP address
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf "My IP address is %s\n" "$_IP"
fi

# HDMI off
tvservice -o

exit 0
```

結局、有効な解決方法が見つかるまでの回避策として「4」にWiFiのUSBドングルを刺して再起動したら、劇的にWiFiの接続性は改善した。(自動的にUSBドングルを認識するので、特に設定などを変える必要はない)

なお、この際、このままでも、特に問題ないようだが、wlan0,wlan1の２つにIPアドレスが割り振られるのでオンボードのwlan0を無効化して、割り振られるIPアドレスを１つにしたほうがスッキリする。それには以下を実行する：  
wlan0無効化：
```bash

sudo iwconfig wlan0 txpower off
sudo reboot
```
もとに戻すには以下を実行する：  
wlan0有効化：
```bash

sudo iwconfig wlan0 txpower auto
sudo reboot
```

参考URL:   
[Raspberry Pi で HDMI 接続のディスプレイをオン・オフ](https://blog.withachristianwife.com/2018/12/20/control-hdmi-display-with-pi/)   
　  

以上
