
2021/3/6  
初版

import platformio to Arduino-IDE
# import platformio to Arduino-IDE

## 概要
platformioのプログラムをArduino-IDEにインポートする方法について述べる。  
以下を前提とする：  
(1)platformioのframeworkとして、arduinoに設定している   
(2)Arduino-IDEは、インストール済みで使用するボードのボードマネージャーでの設定が完了している  
(3)Arduino-IDEのディレクトリ構成はデフォルトのままとする   
(4)ホストPCのOSとしてubuntuを使用している    

インポート作業の流れとしては以下になる：   
(1)platformioのプログラムをArduinoのディレクトリにコピーする  
(2)platformioで使用しているライブラリをArduino/librariesにコピーする  

## 実際の手順
ここでは例として、platformioのwiot-nixie-NTP(ディレクトリ)のプログラムを
Arduino-IDEにインポートすることにする。
```

cd pio_ws
# プログラムをコピーする
cp -r wiot-nixie-NTP/src ~/Arduino/wiot-nixie-NTP
mv ~/Arduino/wiot-nixie-NTP/*.ino ~/Arduino/wiot-nixie-NTP/wiot-nixie-NTP.ino

cd wiot-nixie-NTP
# ライブラリをダウンロードする
# (以前、ビルドしたことがあれば不要)
pio run
# ライブラリをコピーする
cp -r .pio/libdeps/seeed_wio_terminal/* ~/Arduino/libraries/
```
ここで「seeed_wio_terminal」は、ボード名称になるので、自分の状況にあわせて変更すること。

## 参考情報

スケッチ関連：  
[nixieクロックにＮＴＰクライアントの機能を追加する(V2)](https://beta-notes.way-nifty.com/blog/2021/02/post-2082bc.html)    
[Wio-Terminal/ESP8622/ESP32ボードを共通のスケッチで動かす(NTP-CLIENT編)](https://beta-notes.way-nifty.com/blog/2020/08/post-3484c0.html) 

PlatformIO関連:   
[arduinoフレームワーク用platformio.ini集](https://beta-notes.way-nifty.com/blog/2021/02/post-2b331d.html)   
[windows10にplatformioをインストールする(scoop版)](https://beta-notes.way-nifty.com/blog/2021/02/post-173bac.html)   
[WSL2でのplatformioやPico-SDKの利用方法](https://beta-notes.way-nifty.com/blog/2021/02/post-72506a.html)   
[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

Arduino-IDE関連：  
[Arduino IDEをUbuntu 20.04にインストールする](https://qiita.com/ma2shita/items/927aa9e55962b4137518)  
[How to add boards in the board manager](https://support.arduino.cc/hc/en-us/articles/360016119519-How-to-add-boards-in-the-board-manager)  
[Customizing the Arduino IDE](https://playground.arduino.cc/Main/CustomizeArduinoIDE/)  
[Arduino IDEライブラリのインストールとディレクトリ(フォルダ)構成 (Windows, Mac, Linux対応)](https://www.indoorcorgielec.com/resources/arduinoide%E8%A8%AD%E5%AE%9A/arduino-ide%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%A8%E3%83%87%E3%82%A3%E3%83%AC%E3%82%AF%E3%83%88%E3%83%AA%E6%A7%8B%E6%88%90/)    
[A Tour Inside Arduino Core: Source Files, How to Make A New Core and Arduino Building Steps](https://atadiat.com/en/e-arduino-core-source-files-make-new-core-building-steps/)  

以上
