
2020/9/18:  
初版  

OmegaT install
# OmegaT install

## 概要
翻訳支援ツールOmegaT(linux版)をインストールする

## download
https://omegat.org/download   
上のurlからダウンロードするが自分の環境にあったものをダウンロードする：   
・JREのバージョンの違いで動作しないことがないように「with JRE」が付いているものを選択する。  
・「64-bit」が付いているものと、そうでないものがあるで、自分の環境に合わせて選択する。  
・「Standard Version - OmegaT 4.3.2」と「Latest Version - OmegaT 5.3.0」があるので、どちらかを選択する。  

自分の環境の場合、以下のものを選択した：  
「Linux with 64-bit JRE」(Standard Version - OmegaT 4.3.2)

ダウンロードが終了すると以下のファイルがダウンロードできる：  
OmegaT_4.3.2_Linux_64.tar.bz2

## install
以下を実行してインストールする：
```bash

cc Download
tar -jxvf OmegaT_4.3.2_Linux_64.tar.bz2
cd OmegaT_4.3.2_Linux_64
sudo ./linux-install.sh

```

## 実行
```bash

omegat
```

ubuntuの環境で動かしたが
バージョンが古いとエラーで動作しない。   
(16.04では動作しなかった.インストール状況の問題の可能性あり)   
ubuntu_18.04では正常動作した。


## 参考情報

https://ja.wikipedia.org/wiki/OmegaT   
OmegaT

https://ja.wordpress.org/team/handbook/support/omegat/  
翻訳エディター OmegaT の使用方法

https://omegat.org/ja/howtos/linux   
OmegaT 技術情報：Linux

以上
