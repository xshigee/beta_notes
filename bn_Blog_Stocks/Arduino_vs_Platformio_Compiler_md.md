
2020/7/21  
ヘッダー部分改版  

2020/7/19  
platformio.ini設定追加   

2020/7/14   
初版  

Arduino vs Platformio compiler
# Arduino vs Platformio compiler

## 概要
本記事は「Arduino-IDEとPlatformioのコンパイラーの挙動の違いについて」纏めたものである。
実際に使ってみて気がついたことを記述したので、勘違いや誤解がある可能性はある。
以降、Arduino-IDEのコンパイラをAコンパイラ、PlatformioのコンパイラをPコンパイラとする。

## ファイルタイプ(.ino or .cpp)
Pコンパイラの場合、スケッチのファイルタイプとして、.inoや.cppが使えるが、.cppの場合、スケッチの先頭に「#include <Arduino.h>」を入れる必要がある。ただし、スケッチの関数の定義の順序によってはコンパイル時に「未定義エラー」が出ることがある。この場合、関数定義を先頭のほうへ移動することで、エラーをなくすることができる。なるべく変更せずにコンパイルするには、ファイルタイプを.inoにする。

## ヘッダー
Aコンパイラで正常にコンパイルできたスケッチ(.ino)をPコンパイラでコンパイルした場合、ヘッダーがない旨をエラーがでることがある。この場合は、エラーになったヘッダーをスケッチに追加する。経験的に頻繁にあるのは「<SPI.h>」ヘッダーがない旨のエラーが出る。   
この場合は、スケッチの先頭に「#include <SPI.h>」を追加する。
  
または、platformio.iniに「lib_ldf_mode = deep+」を追加することでも解消できる。  
  
## キャスティング
コンパイラの差というより、バージョンやコンパイルオプションによる差だと思うが、キャストに関するエラーでコンパイルできないことがある。
その場合は、エラーの箇所をキャスティングしてエラーと取る。

例：
```

  oled.print_String(20, 50, "Hello WaveShare !", FONT_5X8);
→
  oled.print_String(20, 50, (const uint8_t *)"Hello WaveShare !", (FONT_SIZE)FONT_5X8); // force casting

```
上の例は以下のソースからの抜粋：  
https://www.waveshare.com/w/upload/e/eb/1.5inch_RGB_OLED_Module_Code.7z   

## platformio.ini　設定追加
Aコンパイラでは正常に動作するがＰコンパイラではコンパイラ・エラーにならないが正常動作しないことがあった。  
この場合、「lib_ldf_mode = deep+」を以下のようにplatformio.iniに追加することにより正常動作するようになった。  
platformio.ini
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

[env:huzzah]
platform = espressif8266
#board = huzzah
board = esp_wroom_02
framework = arduino

lib_ldf_mode = deep+

```
この問題が起きたスケッチ例：  
src/main.ino
```c++


/*
 *  forked from: http://nopnop2002.webcrow.jp/HSES-LCD24/HSES-LCD24-12.html
 *  Simple HTTP get webclient test
 */

#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>

//#include <AtWiFi.h>
//#include <HTTPClient.h>

#include <SPI.h>

#define TAGMAX 10
#define LENMAX 128

const char* ssid = "your_ssid";
const char* password = "your_passwd";

HTTPClient http;

struct TAG {
  uint8_t StartName;
  uint8_t StartValue;
  uint8_t TagPos;
  unsigned char TagName[LENMAX];
  unsigned char TagCurrent[LENMAX];
  uint8_t ValueCount;
  unsigned char TagValue[TAGMAX][LENMAX];
};

struct TAG description;

void httpPrint(int len, uint8_t *buff) {
  char ch[2];
  ch[1] = 0;
  for(int i=0; i<len; i++) {
    ch[0] = buff[i];
    if (isAlphaNumeric(ch[0])) {
      Serial.printf("%c",ch[0]);
    } else if (strstr("\"!#$%&'()=-<> ;:/,.@[]+*",ch)) {
      Serial.printf("%c",ch[0]);
    } else if (ch[0] == 0x0a) {
      //Serial.printf("\n");
      Serial.printf("\r\n");
    } else {
//      Serial.printf("0x%02x",ch[0]);
      Serial.printf("*");
    }
  }
}

void initTag(struct TAG* hoge,char* name) {
   hoge->StartName=0;
   hoge->StartValue=0;
   hoge->TagPos=0;
   strcpy((char *)hoge->TagName,name);
   memset(hoge->TagCurrent,0,sizeof(hoge->TagCurrent));
   hoge->ValueCount=0;
   memset(hoge->TagValue,0,sizeof(hoge->TagValue));
}

void addTagCurrent(struct TAG* hoge, char ch) {
   int pos;
   pos=hoge->TagPos;
   if (pos == LENMAX-1) return;
   hoge->TagCurrent[pos]=ch;
   hoge->TagCurrent[++pos]=0;
   (hoge->TagPos)++;
}

void startTag(struct TAG* hoge) {
   hoge->StartName=1;
   hoge->TagPos=0;
   if (hoge->StartValue) {
     hoge->StartValue=0;
     (hoge->ValueCount)++;
   }
}

void endTag(struct TAG* hoge) {
   hoge->StartName=0;
   hoge->TagPos=0;
   if (strcmp ((char *)hoge->TagName, (char *)hoge->TagCurrent) == 0) {
     if (hoge->ValueCount == TAGMAX) return;
     hoge->StartValue=1;
   }
}

void addTagValue(struct TAG* hoge, char ch) {
   int pos;
   int count;
   pos=hoge->TagPos;
   count=hoge->ValueCount;
   if (pos == LENMAX-1) return;
   hoge->TagValue[count][pos]=ch;
   hoge->TagValue[count][++pos]=0;
   (hoge->TagPos)++;
}

void dumpTag(struct TAG hoge) {
   int i;
   Serial.printf("\r\nTagName=%s\r\n",hoge.TagName);
   Serial.printf("TagCurrent=%s\r\n",hoge.TagCurrent);
   Serial.printf("ValueCount=%d\r\n",hoge.ValueCount);
   for(i=0;i<hoge.ValueCount;i++){
   Serial.printf("TagValue[%d]=%s\r\n",i,hoge.TagValue[i]);
   }
}

void httpParse(int len, uint8_t *buff, struct TAG *hoge) {
   int i;
   char ch;
   for(i=0;i<len;i++) {
     ch=buff[i];
     if (ch == '<') {
       startTag(hoge);
     } else if (ch == '>') {
       endTag(hoge);
     } else {
       if (hoge->StartName) addTagCurrent(hoge,ch);
       if (hoge->StartValue) addTagValue(hoge,ch);
     }
   }
}

void setup() {
  int httpCode;
  //delay(1000);Serial.begin(9600);
  delay(1000);Serial.begin(115200);

  // start by connecting to a WiFi network
  Serial.println();
  Serial.println();
  Serial.print("Wait for WiFi...");
  WiFi.begin(ssid, password);
 
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
 
  Serial.println("");
  Serial.println("WiFi connected");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());

  initTag(&description,"description");
  dumpTag(description);

  Serial.printf("[HTTP]begin..\r\n");
  //http.begin("weather.livedoor.com", 80, "/forecast/rss/area/230010.xml");
  http.begin("weather.livedoor.com", 80, "/forecast/rss/area/130010.xml"); // Tokyo
  //http.begin("weather.livedoor.com", 80, "/forecast/rss/area/270000.xml"); // Osaka
  //http.begin("weather.livedoor.com", 80, "/forecast/rss/area/140010.xml"); // Yokohama

  httpCode = http.GET();
  if(httpCode > 0){
    Serial.printf("[HTTP] GET ... code: %d\r\n",httpCode);
    if(httpCode == HTTP_CODE_OK) {
      Serial.printf("[HTTP] OK\r\n");
      int len = http.getSize();
      uint8_t buff[128] = {0};
      WiFiClient *stream = http.getStreamPtr();
      while(http.connected() && (len > 0 || len == -1)){
        size_t size = stream->available();
        if(size){
          if(size > sizeof buff) size = sizeof buff;
          int c = stream->readBytes(buff,size);
//          httpPrint(c,buff);
//          delay(1);
          httpParse(c,buff,&description);
        }
      }
      Serial.printf("[HTTP] Disconnected\r\n");
      dumpTag(description);
    } else {
      Serial.printf("[HTTP] Server response Not OK\r\n");
    }
  } else {
    Serial.printf("[HTTP] GET... failed, error: %s\r\n", http.errorToString(httpCode).c_str());
  }
  http.end();
}

void loop() {
}
```

## 参考情報

「lib_ldf_mode = deep+」の解説：   
https://docs.platformio.org/en/latest/librarymanager/ldf.html#ldf   

[開発ツールPlatformIOをcliで使う(Seeeduino-Wio-Terminal/Arduino版)](https://beta-notes.way-nifty.com/blog/2020/07/post-3f0abb.html)   
[開発ツールPlatformIOをcliで使う(Seeeduino-XIAO版)](https://beta-notes.way-nifty.com/blog/2020/06/post-6f19ad.html)   
[ESP-WROOM-02ボードでMQTTとMPL3115A2(気圧・高度・気温センサ)を動かす（Arduino版）](https://beta-notes.way-nifty.com/blog/2020/07/post-db9550.html)   

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  


以上

