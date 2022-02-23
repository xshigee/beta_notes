
2020/3/4

Raspberry Pi Zero_W Serial
# Raspberry Pi Zero_W Serial

## 概要
Raspberry_Pi_Zero-W(以下、Rpi)にコンソール用シリアルを接続する際の設定について記する。

## 接続
以下の表のようにRaspberry_PiのピンとUSBシリアル(3.3V)を接続する。

接続表

| Rpi_Pin | USB-serial |
| :--: | :----: |
| p06(GND) | GND |
| p08(TX) | RXD |
| p10(RX) | TXD |

## 設定/起動
(1)ブートSDのbootのなかにあるconfig.txtの内容の末尾に以下を追加する。
```bash

enable_uart=1
```
最新版(buster)では上の設定のみで良いようだ。
(古いバージョンでは色々設定する必要があるようだ)
他のRpi(初代、2,3,4)であっても、最新版であれば、
この設定で動作するようだ。  
(2)上で修正したブートSDをRpiに刺す。  
(3)RpiをUSBシリアルで接続する。  
(4)ホストPC側のシリアル通信を起動しておく。  

起動例：  
```bash

picocom /dev/ttyUSB0 -b115200

```
(5)Rpiの電源を入れる。  
(6)通信画面にログイン・プロンプトが表示されたら
ユーザー名、パスワードを入力してログインする。


## 所感
WiFi環境でsshログインしても
接続が不安定でストレスが貯まることが多い場合は、
USBシリアルを接続する面倒臭さを乗り越えて
シリアルでログインしてしまえば、接続の不安定が解消して
快適に使用できる。


## 参考情報

[ＦＴＤＩ　ＵＳＢシリアル変換ケーブル（３．３Ｖ）](http://akizukidenshi.com/catalog/g/gM-05840/)  
[Raspberry Pi Pinout/UART](https://pinout.xyz/pinout/uart)   

以上
