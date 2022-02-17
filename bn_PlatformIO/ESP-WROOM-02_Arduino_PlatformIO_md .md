
2020/1/6

ESP-WROOM-02 tool (VScode+PlatformIO)
# ESP-WROOM-02 tool (VScode+PlatformIO)

## 概要
「ESP-WROOM-02 Arduino互換ボード」の開発ツールとして、VScodeの拡張としてPlatformIOを導入して、そのなかで開発ツール(コンパイラ、リンカ、アップローダー)をインストールする。
ここでは、linux環境でのインストール方法について説明する。   


## 準備
ツールをインストール前に環境整備として以下を設定する：  
(1)Python 2.7またはPython 3.5+のインストール
```bash

sudo apt-get install python3
```
(2)udevのruleの設定
```bash

curl -fsSL https://raw.githubusercontent.com/platformio/platformio-core/master/scripts/99-platformio-udev.rules | sudo tee /etc/udev/rules.d/99-platformio-udev.rules
sudo service udev restart
sudo usermod -a -G dialout $USER
sudo usermod -a -G plugdev $USER/home/komatsu/Documents/PlatformIO/Projects/LPC1768_mbed_01/src/main.cpp
```

## 開発ツールのインストール
上の準備が終わったら、プロセッサとボードの種類は異なるが、インストール方法が分かりやすいので、以下を参考にインストールする。：      
https://www.media.lab.uec.ac.jp/?page_id=1414  
ESP32をVSCodeとPlatformIO IDEで動かす方法   

新しいプロジェクトを作り、Boardとして、   
「ESP-WROOM-02」を選択し、Frameworkとして   
「arduino」を選択して
Nameに任意の名前を入力して[Finish]をクリックする。   
(このNameはプロジェクト名かつフォルダ名になる)

その後、必要なツールのインストールが開始するが、完了後、platform.iniの内容に以下が含まれることを確認する。   
```

platform = espressif8266
board = esp_wroom_02
framework = arduino
```

## IDE設定
platformio.iniファイルを以下のように編集する(デフォルトにupload_protocolを追加する)：  
```

;PlatformIO Project Configuration File
;
;   Build options: build flags, source filter
;   Upload options: custom upload port, speed and extra flags
;   Library options: dependencies, extra library storages
;   Advanced options: extra scripting
;
; Please visit documentation for the other options and examples
; https://docs.platformio.org/page/projectconf.html

[env:esp_wroom_02]
platform = espressif8266
board = esp_wroom_02
framework = arduino

upload_protocol = esptool

```

## ビルドテストのためのサンプルプログラム
サンプル(#defineで三つのスケッチが切り替えられる)：   
```cpp

#include <Arduino.h>

//#define BLINKY
#define ASCII_TABLE
//#define WIFI_TEST

#ifdef BLINKY
/*
  Blink

  Turns an LED on for one second, then off for one second, repeatedly.

  Most Arduinos have an on-board LED you can control. On the UNO, MEGA and ZERO
  it is attached to digital pin 13, on MKR1000 on pin 6. LED_BUILTIN is set to
  the correct LED pin independent of which board is used.
  If you want to know what pin the on-board LED is connected to on your Arduino
  model, check the Technical Specs of your board at:
  https://www.arduino.cc/en/Main/Products

  modified 8 May 2014
  by Scott Fitzgerald
  modified 2 Sep 2016
  by Arturo Guadalupi
  modified 8 Sep 2016
  by Colby Newman

  This example code is in the public domain.

  http://www.arduino.cc/en/Tutorial/Blink
*/

#define LED_BULTIN 4 // external LED

// the setup function runs once when you press reset or power the board
void setup() {
  // initialize digital pin LED_BUILTIN as an output.
  pinMode(LED_BUILTIN, OUTPUT);
}

// the loop function runs over and over again forever
void loop() {
  digitalWrite(LED_BUILTIN, HIGH);   // turn the LED on (HIGH is the voltage level)
  delay(100);                       // wait for a second
  digitalWrite(LED_BUILTIN, LOW);    // turn the LED off by making the voltage LOW
  delay(100);                       // wait for a second
}
#endif
//=========================================================================

#ifdef ASCII_TABLE
/*
  ASCII table

  Prints out byte values in all possible formats:
  - as raw binary values
  - as ASCII-encoded decimal, hex, octal, and binary values

  For more on ASCII, see http://www.asciitable.com and http://en.wikipedia.org/wiki/ASCII

  The circuit: No external hardware needed.

  created 2006
  by Nicholas Zambetti <http://www.zambetti.com>
  modified 9 Apr 2012
  by Tom Igoe

  This example code is in the public domain.

  http://www.arduino.cc/en/Tutorial/ASCIITable
*/

void setup() {
  //Initialize serial and wait for port to open:
  Serial.begin(9600);
  while (!Serial) {
    ; // wait for serial port to connect. Needed for native USB port only
  }

  // prints title with ending line break
  Serial.println("ASCII Table ~ Character Map");
}

// first visible ASCIIcharacter '!' is number 33:
int thisByte = 33;
// you can also write ASCII characters in single quotes.
// for example, '!' is the same as 33, so you could also use this:
// int thisByte = '!';

void loop() {
  // prints value unaltered, i.e. the raw binary version of the byte.
  // The Serial Monitor interprets all bytes as ASCII, so 33, the first number,
  // will show up as '!'
  Serial.write(thisByte);

  Serial.print(", dec: ");
  // prints value as string as an ASCII-encoded decimal (base 10).
  // Decimal is the default format for Serial.print() and Serial.println(),
  // so no modifier is needed:
  Serial.print(thisByte);
  // But you can declare the modifier for decimal if you want to.
  // this also works if you uncomment it:

  // Serial.print(thisByte, DEC);


  Serial.print(", hex: ");
  // prints value as string in hexadecimal (base 16):
  Serial.print(thisByte, HEX);

  Serial.print(", oct: ");
  // prints value as string in octal (base 8);
  Serial.print(thisByte, OCT);

  Serial.print(", bin: ");
  // prints value as string in binary (base 2) also prints ending line break:
  Serial.println(thisByte, BIN);

  // if printed last visible character '~' or 126, stop:
  if (thisByte == 126) {    // you could also use if (thisByte == '~') {
    // This loop loops forever and does nothing
    while (true) {
      continue;
    }
  }
  // go on to the next character
  thisByte++;
}
//================================================
#endif

#ifdef WIFI_TEST

#include <ESP8266WiFi.h>
 
//ご自分のルーターのSSIDを入力してください
//方式は、WPAかWPA2を使用すること(WEPは接続できないようだ)
//const char* ssid = "xxxx";
//ご自分のルーターのパスワード
//const char* password = "xxxx";
 
boolean Ini_html_on = false;//ブラウザからの初回HTTPレスポンス完了したかどうかのフラグ
 
WiFiServer server(80);
WiFiClient client;
 
void setup() {
  //Serial.begin(115200);
  Serial.begin(9600); // PlatformIO Serial Monitor Speed
  // Connect to WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
 
  WiFi.begin(ssid, password);
 
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");
 
  // Start the server
  server.begin();
  Serial.println("Server started");
 
  // Print the IP address
  Serial.println(WiFi.localIP());
}

void Ini_HTTP_Response()
{
  client = server.available();//クライアント生成
  delay(1);
  String req;
 
  while(client){
    if(client.available()){
      req = client.readStringUntil('\n');
      Serial.println(req);
      if (req.indexOf("GET / HTTP") >= 0 || req.indexOf("GET /favicon") >= 0){//ブラウザからリクエストを受信したらこの文字列を検知する
        //Google Chromeの場合faviconリクエストが来るのでそれも検出する
        Serial.println("-----from Browser FirstTime HTTP Request---------");
        Serial.println(req);
        //ブラウザからのリクエストで空行（\r\nが先頭になる）まで読み込む
        while(req.indexOf("\r") != 0){
          req = client.readStringUntil('\n');//\nまで読み込むが\n自身は文字列に含まれず、捨てられる
          Serial.println(req);
        }
        req = "";
        delay(10);//10ms待ってレスポンスをブラウザに送信
 
        //メモリ節約のため、Fマクロで文字列を囲う
        //普通のHTTPレスポンスヘッダ
        client.print(F("HTTP/1.1 200 OK\r\n"));
        client.print(F("Content-Type:text/html\r\n"));
        client.print(F("Connection:close\r\n\r\n"));//１行空行が必要
        //ここからブラウザ表示のためのHTML吐き出し
        client.println(F("<!DOCTYPE html>"));
        client.println(F("<html>"));
        client.println(F("<body>"));
        client.println(F("<font size=30>"));
        client.println(F("Hello World"));
        client.println(F("</font>"));
        client.println(F("</body>"));
        client.println(F("</html>"));
 
        delay(10);//これが重要！これが無いと切断できないかもしれない。
        client.stop();//一旦ブラウザとコネクション切断する。
        delay(10);
        Serial.println("\nGET HTTP client stop--------------------");
        req = "";
        Ini_html_on = false;  //一回切りの接続にしたい場合、ここをtrueにする。                
      }
    }
  }
}

void loop() {
  if(Ini_html_on == false){
      Ini_HTTP_Response();
  }else if(client.available()){
    Serial.print(client.read());
  }
  delay(1);//これは重要かも。
}

#endif
```

## build/upload
(1)ボードとホストPCをUSBケーブルで接続する。   
(2)VScode画面の最下行にあるアイコンからBuildのアイコンをクリックしてビルド実行する。   
(3)VScode画面の最下行にあるアイコンからUploadのアイコンをクリックしてアップロード(ボードへのプログラム書込)を実行する。    
(4)書き込み終了後、書き込んだプログラムが起動する。   
uploadが上手く行かない時は、USBケーブルを挿し直して再度uploadを実行する。   
ボードのシリアル出力を見たい場合、VScode画面の最下行にある[Serial Monitor]をクリックしてシリアル画面に切り替える。切り替え後、ボードの[reset]ボタンを押す。

build出力ログ例：
```
> Executing task in folder WROOM02_arduino_01: platformio run <

Processing esp_wroom_02 (platform: espressif8266; board: esp_wroom_02; framework: arduino)
------------------------------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/espressif8266/esp_wroom_02.html
PLATFORM: Espressif 8266 2.3.2 > ESP-WROOM-02
HARDWARE: ESP8266 80MHz, 80KB RAM, 2MB Flash
PACKAGES: toolchain-xtensa 2.40802.191122 (4.8.2), tool-esptool 1.413.0 (4.13), tool-esptoolpy 1.20800.0 (2.8.0), framework-arduinoespressif8266 2.20603.191216 (2.6.3)
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 29 compatible libraries
Scanning dependencies...
Dependency Graph
|-- <ESP8266WiFi> 1.0
Building in release mode
Retrieving maximum program size .pio/build/esp_wroom_02/firmware.elf
Checking size .pio/build/esp_wroom_02/firmware.elf
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
DATA:    [===       ]  32.9% (used 26940 bytes from 81920 bytes)
PROGRAM: [===       ]  25.0% (used 261252 bytes from 1044464 bytes)

===== [SUCCESS] Took 1.72 seconds =====

Terminal will be reused by tasks, press any key to close it.
```

Upload出力ログ例：
```
> Executing task in folder WROOM02_arduino_01: platformio run --target upload <

Processing esp_wroom_02 (platform: espressif8266; board: esp_wroom_02; framework: arduino)
-----------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/espressif8266/esp_wroom_02.html
PLATFORM: Espressif 8266 2.3.2 > ESP-WROOM-02
HARDWARE: ESP8266 80MHz, 80KB RAM, 2MB Flash
PACKAGES: toolchain-xtensa 2.40802.191122 (4.8.2), tool-esptool 1.413.0 (4.13), tool-mkspiffs 1.200.0 (2.0), tool-esptoolpy 1.20800.0 (2.8.0), framework-arduinoespressif8266 2.20603.191216 (2.6.3)
LDF: Library Dependency Finder -> http://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 29 compatible libraries
Scanning dependencies...
Dependency Graph
|-- <ESP8266WiFi> 1.0
Building in release mode
Retrieving maximum program size .pio/build/esp_wroom_02/firmware.elf
Checking size .pio/build/esp_wroom_02/firmware.elf
Advanced Memory Usage is available via "PlatformIO Home > Project Inspect"
DATA:    [===       ]  33.3% (used 27276 bytes from 81920 bytes)
PROGRAM: [===       ]  26.2% (used 273564 bytes from 1044464 bytes)
Configuring upload protocol...
AVAILABLE: espota, esptool
CURRENT: upload_protocol = esptool
Looking for upload port...
Auto-detected: /dev/ttyUSB0
Uploading .pio/build/esp_wroom_02/firmware.bin
esptool.py v2.8
Serial port /dev/ttyUSB0
Connecting....
Chip is ESP8266EX
Features: WiFi
Crystal is 26MHz
MAC: 18:fe:34:ef:0d:18
Uploading stub...
Running stub...
Stub running...
Configuring flash size...
Auto-detected Flash size: 4MB
Flash params set to 0x0040
Compressed 277712 bytes to 202278...
Wrote 277712 bytes (202278 compressed) at 0x00000000 in 18.0 seconds (effective 123.6 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...

===== [SUCCESS] Took 21.63 seconds =====

Terminal will be reused by tasks, press any key to close it.
```


WIFI_TEST実行中のシリアル画面(例)　   
ボードの[reset]ボタンを押した後：   
```

platformio device monitor --baud 9600
--- Miniterm on /dev/ttyUSB0  9600,8,N,1 ---
--- Quit: Ctrl+C | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H ---
Connecting to xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
..
WiFi connected
Server started
192.168.0.27

```
この例では、192.168.0.27にブラウザーでアクセスすると
「Hello World」が表示される。


## フォルダ構成
デフォルトの設定では、home/Documents/PlatformIO/Projectsのなかにプロジェクト名のフォルダが生成される。

## 参考情報  

[A new generation ecosystem for embedded development](https://platformio.org/)    
[ESP-WROOM-02 Arduino互換ボード](https://www.switch-science.com/catalog/2620/)   
[ESP-WROOM-02 Arduino互換ボードのデジタルI/Oの番号について](http://trac.switch-science.com/wiki/ESP-WROOM-02_arduino_dio)   
[ESP8266, ESP-WROOM-02, ESPr Developer トラブルシューティングまとめ](https://www.mgo-tec.com/blog-entry-esp8266-wroom-02-espr-developer-trouble-shooting.html)   

WIFI_TESTのプログラムは以下のURLのプログラムを流用した：   
https://www.mgo-tec.com/blog-entry-ss-wroom-howto01.html   



以上
