
2020/8/7:  
AE-ESP-WROOM02-DEV,ACアダプターについて追加する、

2020/7/8：  
初版  

XIAO WiFi module Test
# XIAO WiFi module Test

## 概要
XIAOを使って以下のWiFiモジュール(ESP-WROOM-02)の動作確認をする(Seeeduino-XIAO版)。  
開発ツールのインストールついては[[開発ツールPlatformIOをcliで使う(Seeeduino-XIAO版)](https://beta-notes.way-nifty.com/blog/2020/06/post-6f19ad.html)]を参照のこと。(ホストPCとしてはubuntuを想定している)

・[ＥＳＰ－ＷＲＯＯＭ－０２ピッチ変換キット](http://akizukidenshi.com/catalog/g/gK-13646/)  
・[Ｗｉ－Ｆｉモジュール　ＥＳＰ－ＷＲＯＯＭ－０２　ＤＩＰ化キット](http://akizukidenshi.com/catalog/g/gK-09758)  
または   
・[ＥＳＰ－ＷＲＯＯＭ－０２開発ボード(AE-ESP-WROOM02-DEV)](http://akizukidenshi.com/catalog/g/gK-12236/)   
このボードには出荷時にATファームウェアが書き込まれている。(WiFiモジュールとして使用する場合、Arduinoのファームなどを書き込まない)


## 接続
| ESP-WROOM-02 | XIAO |
| ---: | :--- |
| 3V3 | 3V3 |
| EN | PullUp(10kΩ)|
| IO15 |　PullDown(１0kΩ)|
| IO2 |PullUp(10kΩ)|
| IO0 |PullUp(10kΩ)|
| GND | GND |
| RST | NC(リセット時はいったんGNDにする)|
| TXD | D7(RX) |
| RXD | D6(TX) |

注意： ESP-WROOM-02の消費電力が多くて、XIAOの電源容量でまかないきれず
誤動作するので、補助電源が必要となる。簡易的なやり方としては、
もうひとつのXIAOを電源が代わりにして、そのXIAOの3V3/GNDを
ESP-WROOM-02に接続する。（このとき、/dev/ttyACMxが出現しないように、
たんなる電源用のUSB出力に接続する）  


ＥＳＰ－ＷＲＯＯＭ－０２開発ボード(AE-ESP-WROOM02-DEV)の場合
| AE-ESP-WROOM02-DEV | XIAO |
| ---: | :--- |
| 1(3V3入力)|　USB給電の場合、接続しない |
| 14(TXD) | D7(RX) |
| 13(RXD) | D6(TX) |
| 9(GND) | GND |

USB給電の場合は、オンボードのUSBにホストＰＣなどを接続して行なう。（USBシリアルとしては使用しない）
XIAOのUSBシリアルして使用するので、XIOAのUSBはホストPCに接続して通信ソフトを起動する。  
ＥＳＰ－ＷＲＯＯＭ－０２開発ボードの場合、外付け部品が不要で、電源も安定しているので、お薦めである。


## XIAOスケッチ(USBserial)
USBシリアルとして動作するプログラム：   
src/USBserial.ino
```c++

void setup() {
  Serial.begin(115200);
  Serial1.begin(115200);
}

void loop() {
  if (Serial.available()) {
    char c = (char)Serial.read();
    Serial1.write(c);
  }

  if (Serial1.available()) {
    char c = (char)Serial1.read();
    Serial.write(c);
  }
}
```
D6(TX),D7(RX)を相手のRX,TXに、それぞれ接続する。(GND同士も接続する)

## WiFi動作確認用コマンド例

「picocom /dev/ttyACM0 -b115200」で通信ソフトを起動して
以下のコマンド列を入力する。
```
// get firmware version 
AT+GMR
 
AT+CWQAP

AT+CWMODE=1

AT+CWJAP="your_ssid","your_passwd"

// IP address
AT+CIFSR

AT+CIPMUX=1

AT+CIPSERVER=1,80
  
// telnet access
AT+CIPSTART=1,"TCP","towel.blinkenlights.nl",23
```
//で始まる行はコメントなので、入力しない。
その他の行は、通信ソフトの画面にコピー&ペーストして
行末は、[Enter]+[Contorl+J]を入力する。

## 出力例(レスポンス)

```

picocom /dev/ttyACM0 -b115200


AT+GMR

AT version:1.3.0.0(Oct 28 2016 11:29:39)
SDK version:2.0.0(6ccde98)
compile time:Oct 28 2016 13:18:18
OK
AT+CWQAP


OK
WIFI DISCONNECT
AT+CWMODE=1


OK
AT+CWJAP="your_ssid","your_passwd"

WIFI CONNECTED
WIFI GOT IP

OK
AT+CIFSR

+CIFSR:STAIP,"192.168.0.2"
+CIFSR:STAMAC,"5c:cf:7f:eb:80:17"

OK
AT+CIPMUX=1


OK
AT+CIPSERVER=1,80


OK
AT+CIPSTART=1,"TCP","towel.blinkenlights.nl",23

1,CONNECT

OK

#以下、startwarsのASCIIアートアニメが表示される。(未処理なのでヘッダー(+IPD...)も表示される)
+IPD,1,6:[H[J
+IPD,1,988:[H





                                                                         
      Original Work   : Simon Jansen ( http://www.asciimation.co.nz/ )   
      Telnetification : Sten Spans ( http://blinkenlights.nl/ )          
      Terminal Tricks : Mike Edwards (pf-asciimation@mirkwood.net)       
                                                                         
      The hard work was done by Simon and Mike,                          
      I just placed it online in a different format.                     
                                                                         
      So long And Thanks for all the fish                                
                                                                         
      Sten (I just need a Hug)                                           
                             

...
<省略>
...
#比較的長時間続くので、[Control-A][Control-X]で通信ソフトから抜ける

#モジュールをリセットする場合、「AT+RST」を入力する。リセット後、firmwareの状況を示すログが出力される。
```

リセット後の出力ログの例：
```
AT+RST


OK

 ets Jan  8 2013,rst cause:1, boot mode:(3,3)

load 0x40100000, len 2408, room 16 
tail 8
chksum 0xe5
load 0x3ffe8000, len 776, room 0 
tail 8
chksum 0x84
load 0x3ffe8310, len 632, room 0 
tail 8
chksum 0xd8
csum 0xd8

2nd boot version : 1.6
  SPI Speed      : 40MHz
  SPI Mode       : QIO
  SPI Flash Size & Map: 16Mbit(512KB+512KB)
jump to run user1 @ 1000
ESP8266
AT Instruction Set
#以下、ビットレートが変わるのでゴミになる
xxxìlìxrllxxrllxxr

```


## 次のステップ
ATコマンドが動作することが確認できたので次のステップとしては、XIAOのスケッチを書き変えて、(PCからのATコマンドではなく)スタンドアロンでXIAOでWiFiを制御することになる。

## ATファームウェアのアップデート
特にアップデートせずども使用できるが、アップデートする場合は以下の方法がある：  

オフラインのツールでアップデートできるが
以下のATコマンドでネットワーク経由でアップデートできる：  
ATコマンド：  
```

// get firmware version 
AT+GMR
 
AT+CWQAP

AT+CWMODE=1

AT+CWJAP="your_ssid","your_passwd"

// IP address
AT+CIFSR

// firmaware update
AT+CIUPDATE

```

## 出力例(レスポンス)

```

picocom /dev/ttyACM0 -b115200

OK
AT+GMR
AT version:1.3.0.0(Oct 28 2016 11:29:39)
SDK version:2.0.0(6ccde98)
compile time:Oct 28 2016 13:18:18
OK
AT+CWQAP

OK
WIFI DISCONNECT
AT+CWMODE=1

OK
AT+CWJAP="your_ssid","your_passwd"
WIFI CONNECTED
WIFI GOT IP

OK
AT+CIFSR
+CIFSR:STAIP,"192.168.0.2"
+CIFSR:STAMAC,"5c:cf:7f:eb:80:17"

OK
AT+CIUPDATE
+CIPUPDATE:1  # found server
+CIPUPDATE:2  # connect server
+CIPUPDATE:3  # got edition
+CIPUPDATE:3  # got edition
+CIPUPDATE:4  # start start

OK
#このあと自動的にブートする

```

アップデート後のバージョンは以下となる：
```

AT+GMR
AT version:1.7.4.0(May 11 2020 19:13:04)
SDK version:3.0.4(9532ceb)
compile time:May 27 2020 10:12:20
Bin version(Wroom 02):1.7.4
OK
```

## 追記
[ＥＳＰ－ＷＲＯＯＭ－０２ピッチ変換キット]のほうは3V3-GND間にコンデンサが入っているが
[Ｗｉ－Ｆｉモジュール　ＥＳＰ－ＷＲＯＯＭ－０２　ＤＩＰ化キット]のほうは入っていない。
そのせいか[ＥＳＰ－ＷＲＯＯＭ－０２ピッチ変換キット]のほうが安定して動作するような気がする。
したがって[Ｗｉ－Ｆｉモジュール　ＥＳＰ－ＷＲＯＯＭ－０２　ＤＩＰ化キット]を使用の際、不安定のようなら
3V3-GNDの間にコンデンサ(100uFくらい？)を入れるほうが良いかもしれない。   
参考だが、wio-lite-RISC-VのWiFiモジュールの回路には3V3-GNDの間に、100nFと10uFが
パラレルに入っている。

電源供給として、一番、確実なのは以下のようなACアダプターで電源供給することである。  
・[超小型スイッチングＡＣアダプター３．３Ｖ２Ａ　１００Ｖ～２４０Ｖ　ＧＦ１２－ＵＳ０３３２０](http://akizukidenshi.com/catalog/g/gM-02212/)  
・[ブレッドボード用ＤＣジャックＤＩＰ化キット](http://akizukidenshi.com/catalog/g/gK-05148/)  


## 参考情報

[AE-ESP-WROOM-02-DEV 取説](http://akizukidenshi.com/download/ds/akizuki/AE-ESP-WROOM02-DEV.pdf)  

[XC6206 datasheet](https://www.torexsemi.com/file/xc6206/XC6206.pdf)  
XIAOに載っている電源ICのデータシート   
(最大200mA流せるようだ)

[ESP8266 - AT Instruction Set](https://www.espressif.com/sites/default/files/documentation/4a-esp8266_at_instruction_set_en.pdf)

[ESP-WROOM-02 - ATコマンド集](http://nonnoise.github.io/ESP-WROOM-02/ATcommand.html)  

[Seeeduino XIAO](http://akizukidenshi.com/catalog/g/gM-15178/)   

[XIAO Schematic](https://files.seeedstudio.com/wiki/Seeeduino-XIAO/res/Seeeduino-XIAO-v1.0-SCH-191112.pdf)  
[XIAO Pinout](https://www.electronics-lab.com/wp-content/uploads/2020/01/Seeeduino-XIAO-pinout.jpg)  

[Arduino Nano Pinout](https://components101.com/sites/default/files/component_pin/Arduino-Nano-Pinout.png)   

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

[Seeeduino XIAO Get Started By Nanase](https://wiki.seeedstudio.com/Seeeduino-XIAO-by-Nanase/)  
[コインサイズ Arduino互換機 Seeeduino XIAO を使ってみた](https://qiita.com/nanase/items/0fed598975c49b1d707e#spi-microsd%E3%82%AB%E3%83%BC%E3%83%89)  

以下、オフラインのアップデート関連URL:  
https://github.com/espressif/esptool  
[esptool – ESP8266およびESP32シリアルブートローダユーティリティ](https://githubja.com/espressif/esptool)  
[ESP-WROOM-02(ESP8266)ファームウェアのアップデート手順](https://qiita.com/dobnezmi/items/ba936fa3695b1dd050ad)  
https://www.espressif.com/sites/default/files/ap/ESP8266_NonOS_AT_Bin_V1.7.4.zip  

以上

