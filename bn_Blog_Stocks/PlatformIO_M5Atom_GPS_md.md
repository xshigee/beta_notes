
2020/9/9:  
スケッチ改良  

2020/9/6:  
初版  

PlatformIO M5Atom GPS kit
# PlatformIO M5Atom GPS kit

## 概要
以下の「ATOMIC GPSキット (M8030-KT)」を使用してみる(M5Atom/Arduino版)    
開発ツールのインストールについては「[M5Atomを開発ツールPlatformIOで使う(M5Atom/Arduino版)](https://beta-notes.way-nifty.com/blog/2020/08/post-3486f7.html)」を参照のこと。
(ホストPCとしてはubuntuを想定している)

[ATOMIC GPSキット (M8030-KT)](https://www.switch-science.com/catalog/6474/)  


## platformio.ini
platformio.iniを以下のように編集する：
```

; PlatformIO Project Configuration File
;
;   Build options: build flags, source filter
;   Upload options: custom upload port, speed and extra flags
;   Library options: dependencies, extra library storages
;   Advanced options: extra scripting
;
; Please visit documentation for the other options and examples
; https://docs.platformio.org/page/projectconf.html

[env:esp32dev]
platform = espressif32
board = m5stick-c
framework = arduino

monitor_speed = 115200

lib_deps =
    # use M5Atom lib
    3113
    # use "FastLED"
    126

lib_ldf_mode = deep+
```

## デモ・スケッチのダウンロード
プロジェクトフォルダで以下を実行する：
```bash

cd src
wget https://raw.githubusercontent.com/m5stack/M5-ProductExampleCodes/master/AtomBase/AtomicGPS/AtomicGPS.ino
wget https://raw.githubusercontent.com/m5stack/M5-ProductExampleCodes/master/AtomBase/AtomicGPS/GPSAnalyse.cpp
wget https://raw.githubusercontent.com/m5stack/M5-ProductExampleCodes/master/AtomBase/AtomicGPS/GPSAnalyse.h
```

## スケッチ修正
ダウンロードしたスケッチAtomicGPS.inoを以下の修正点に対応して変更する：   

修正点:   
(1)コンパイルスイッチでNMEA出力のサポート  
(2)SDに出力するログ（緯度経度など）の精度向上  
(3)M5.disでのコンパイルエラー修正  
(4)USBシリアルの速度設定  
(5)有効な緯度経度情報のみをログする  
(6)ログ出力に簡易なフィルタ処理を施す  

src/AtomicGPS.ino(編集済み) 
```c++  

/*This is an example used SerialBT,you can can view gps data by connecting 
 * to Bluetooth assistant on your mobilephone or Serial Monitor
 * the GPS log will be written to SD card
 * 
 */
// do effective when you want NMEA out
//#define NMEA_OUT

#define deltaLat 0.001
#define deltaLon 0.001

#include "M5Atom.h"
#include "GPSAnalyse.h"
#include <SPI.h>
#include "FS.h"
#include <SD.h>
#include <BluetoothSerial.h>
 
BluetoothSerial SerialBT;
GPSAnalyse GPS;

uint64_t chipid;
char chipname[256];

const char filename[] = "/GPSdata.txt";
File txtFile;

float Lat;
float Lon;
String Utc;

bool writeLog(String filename) {          //Write GPSdata to SDcard
  txtFile = SD.open(filename, FILE_APPEND);
  if(txtFile){
    txtFile.printf("%.5f",Lat);
    txtFile.print(", ");
    txtFile.printf("%.5f",Lon);
    txtFile.print(", ");
/*
    txtFile.print(Lat);
    txtFile.print(", ");
    txtFile.print(Lon);
    txtFile.print(", ");
*/
    txtFile.println(Utc);
    txtFile.close();
  }else{
    return false;
  }
  return true;
}

void setup() {

    M5.begin(true,false,true); 
    chipid = ESP.getEfuseMac();
    sprintf( chipname, "SerialBT_%04X", (uint16_t)(chipid >> 32));
    Serial.printf("Bluetooth: %s\n", chipname);   
    SPI.begin(23,33,19,-1);
    if(!SD.begin(-1, SPI, 40000000)){
      Serial.println("initialization failed!");
    } 
    sdcard_type_t Type = SD.cardType();

	  Serial.printf("SDCard Type = %d \r\n",Type);
	  Serial.printf("SDCard Size = %d \r\n" , (int)(SD.cardSize()/1024/1024));

//    M5.dis.fillpix(0x00004f);
      M5.dis.drawpix(0,0x00004f);

Serial.begin(115200);     
    Serial1.begin(9600,SERIAL_8N1,22,-1);
    SerialBT.begin(chipname);
    GPS.setTaskName("GPS");
    GPS.setTaskPriority(2);
    GPS.setSerialPtr(Serial1);
    GPS.start();
}

#ifdef NMEA_OUT
void loop() {
  if (Serial1.available()) {
    char c = Serial1.read();
    Serial.write(c);
    SerialBT.write(c);
  }
}
#endif

#ifndef NMEA_OUT
float prevLat=0.0;
float prevLon=0.0;
void loop() {
    GPS.upDate();
    Lat = GPS.s_GNRMC.Latitude;
    Lon = GPS.s_GNRMC.Longitude;
    Utc = GPS.s_GNRMC.Utc;
    SerialBT.printf("Latitude= %.5f \r\n",Lat);
    SerialBT.printf("Longitude= %.5f \r\n",Lon);
    SerialBT.printf("DATA= %s \r\n",Utc);
    Serial.printf("Latitude= %.5f \r\n",Lat);
    Serial.printf("Longitude= %.5f \r\n",Lon);
    Serial.printf("DATA= %s \r\n",Utc);
    if ((Lat != 0)&&(Lon != 0)) {
      if ((abs(Lat-prevLat)<deltaLat)&&(abs(Lon-prevLon)<deltaLon)) writeLog(filename); // write them when Lat/Lon effective
      prevLat=Lat; prevLon=Lon;
    };
    delay(1000);
}
#endif
```

NMEA出力したい場合、以下を有効(コメントを外す)にする
```
//#define NMEA_OUT
```

## bluetootシリアル接続
スケッチを実行するとbluetoothデバイスとして、「SerialBT_xxxx」が見えるのでペアリングする。
Andoroidアプリの[Bluetooth Serial Monitor](https://play.google.com/store/apps/details?id=com.csa.btserialmonitor&hl=ja)などをインストールして実行すると、シリアル出力の内容がAndoroidの画面に表示される。

NEMA出力を有効にした場合、Andoroidアプリの[Drogger GPS for DG-PRO1(RW)](https://play.google.com/store/apps/details?id=jp.bizstation.drgps)をインストールして実行してbluetoothシリアルを接続する。
Aｎdroidの画面に測位位置の地図表示や衛星の状態などを表示する。  

ただし、自分の環境(Hauwei P20 lite)では非常に不安定だった。
とりあえず、参考のため使用した設定を明記する（これが有効だったというわけではなく、あくまでも参考のため）
```

・自動再接続試行期間：10sec
・接続オプション：すべてオン(3つ)
・NMEA Message Type: GGA,VTG,RMC,SGA,GSV,GLL有効
```

## GPX変換
NMEA出力が有効でない場合、SDにログデータ(GPSdata.txt)ができるが以下の方法でGPXデータに変換できる。
```

(1)ヘッダーを追加する
最初の行に以下を追加する：
latitude,longitude,utc

(2)緯度経度データの削除
ログそのものだとデータが多いので不要なデータは削除する。
最初の座標を出発点、最後の座標を目的点とみなすので
移動せずに測位した場合、２点のみにするほうが良い。

(3)以下がweb版のコンバータなので、これを使用して変換する:
https://www.gpsvisualizer.com/convert_input
```

## GPXをGoogleMapにインポートする
```
(1)GoogleMapから「メニュー/マイプレイス/マイマップ」を選択して、
表示されている画面の最下行にある「地図を作成」をクリックする。

(2)表示されている「無題の地図」の「無題のレイヤ」の「インポート」をクリックする。

(3)インポートするファイルの選択の画面が表示されるので、その枠「アップロード」にGPXのファイルをドラッグ＆ドロップする。
```

## 参考情報

https://intellectualcuriosity.hatenablog.com/entry/2020/07/01/041715  
ATOM GPS Kit (M8030-KT)でお手軽Bluetooth GPSレシーバーを作った

https://landmarkforchrist.com/ja/2557-huawei-p20-lite-unlock-developer-options-in-android.html   
Huawei P20 Lite Androidの開発者向けオプションのロックを解除

https://akizukidenshi.com/catalog/g/gM-12905/  
ＧＰＳ／ＧＬＯＮＡＳＳ受信機（Ｇａｌｉｌｅｏ／ＢｅｉＤｏｕ可）ｕ‐ｂｌｏｘ　Ｍ８搭載　みちびき３機受信対応

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

以上
