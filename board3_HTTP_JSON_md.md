
2020/8/3:  
初版

board3 HTTP/JSON
# board3 HTTP/JSON

## 概要
Wio-Terminal/ESP8622/ESP32ボードを共通のスケッチで[World Time API](http://worldtimeapi.org/)を使う(WorldTimeAPI編)    
本スケッチは、httpアクセスとJSONライブラリを使用してWorldTimeAPIを利用している。   
(ホストPCとしてはubuntuを想定している)

## 外部ライブラリ(platformioの場合)
以下を実行してライブラリをインストールする：
```bash

# JSON lib インストール
#pio lib install "ArduinoJson"
pio lib install 64


```
または、platformio.iniに以下を追加する：
```

lib_deps =
    # use "ArduinoJson" lib
    64
```


## テスト・スケッチ
  
src/HTTP_JSON.ino
```c++

// select board
////#define WIO_TERMINAL
////#define ESP8266
////#define ESP32
//------------------

// 
// HTTP/JSON access for Wio-Terminal/ESP8266/ESP32
//

#ifdef WIO_TERMINAL
#include <AtWiFi.h>
#endif
#ifdef ESP8266
#include <ESP8266WiFi.h>
#endif
#ifdef ESP32
#include <WiFi.h>
#endif

#include <WiFiClientSecure.h>
#include <ArduinoJson.h>

#define BODY_SIZE 1024
char bodyBuff[BODY_SIZE];

// Allocate the JSON document
StaticJsonDocument<400> doc; // change size if error happen

const char* ssid   = "your_ssid";
const char* passwd = "your_passwd";

// prototype
void ConvertUnixTimeToLocalTime(uint64_t unixtime, uint32_t *pyear, uint8_t *pmonth, uint8_t *pday, uint8_t *phour, uint8_t *pminute, uint8_t *psecond);

char *weekday[] = {"Sun","Mon","Tue","Wed","Thu","Fri","Sat"};
uint32_t year;
uint8_t month, day, hour, minu, sec;

void setup() {
    // Initialize serial port
    Serial.begin(115200);
    while (!Serial) continue;

    Serial.print("Connecting to ");
    Serial.println(ssid);

    WiFi.begin(ssid, passwd);

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    Serial.println("");
    Serial.println("WiFi connected");   
    Serial.println("IP address: ");
    Serial.println(WiFi.localIP());
}

bool Http_Get_access(String host, String url, String argument){
    WiFiClient client;
    client.setTimeout(500);

    const int httpPort = 80;
    const char* host2 = host.c_str();
    if (!client.connect(host2, httpPort)) {
        Serial.println("connection failed");
        return false;
    }
    client.print(String("GET ") + url + "?" + argument + " HTTP/1.1\r\n" +
                        "Host: " + host + "\r\n" + 
                        "User-Agent: ESP8266/1.0\r\n" + 
                        "Connection: close\r\n\r\n");
    unsigned long timeout = micros();
    while (client.available() == 0) {
        if ( micros() - timeout  > 5000000) {
            Serial.println(">>> Client Timeout !");
            client.stop();
            return false;
        }
    }

      Serial.println("--- Header ---");
      while (1) { 
        String line = client.readStringUntil('\n');
        Serial.println(line);
        if (line == "\r") {
          break; // end of header
        }
      }

      //Serial.println("--- Body ---");  // for debug
      int bc = 0;
      while (client.available()) {
        bodyBuff[bc] = client.read(); bc++;
      }
      bodyBuff[bc] = 0;
      //Serial.println(bodyBuff); // display body for debug

      return true;
}

void loop() {

    String host = "worldtimeapi.org";
    String url = "/api/timezone/Asia/Tokyo";
    String argument = "";

    Http_Get_access( host, url, argument);
    Serial.println("accessed BODY:");
    Serial.println(bodyBuff);

    // Deserialize the JSON document
    DeserializationError errorJson = deserializeJson(doc, bodyBuff);

    const char* abbreviation = doc["abbreviation"];
    const char* client_ip = doc["client_ip"];
    const char* datetime = doc["datetime"];
    long day_of_week = doc["day_of_week"];
    long day_of_year = doc["day_of_year"];
    long raw_offset = doc["raw_offset"];
    const char* timezone = doc["timezone"];
    long unixtime = doc["unixtime"];
    const char* utc_datetime = doc["utc_datetime"];
    const char* utc_offset = doc["utc_offset"];
    long week_number = doc["week_number"];

    Serial.println();
    Serial.printf("Deserialized:\r\n");
    Serial.printf("abbreviation:%s\r\n",abbreviation);
    Serial.printf("client_ip:%s\r\n",client_ip);
    Serial.printf("datetime:%s\r\n",datetime);
    Serial.printf("day_of_week:%d\r\n",day_of_week);
    Serial.printf("day_of_year:%d\r\n",day_of_year);
    Serial.printf("raw_offset:%d\r\n",raw_offset);
    Serial.printf("timezone:%s\r\n",timezone);
    Serial.printf("unixtime:%d\r\n",unixtime);
    Serial.printf("utc_datetime:%s\r\n",utc_datetime);
    Serial.printf("utc_offset:%s\r\n",utc_offset);
    Serial.printf("week_number:%d\r\n",week_number);
    Serial.println();

    // display current clock from unixtime
    Serial.printf("Current Time from unixtime:\r\n");
    ConvertUnixTimeToLocalTime(unixtime, &year, &month, &day, &hour, &minu, &sec);
    Serial.printf("%4d/%02d/%02d(%s): %02d:%02d:%02d\r\n",
       year, month, day, weekday[day_of_week], hour, minu, sec);
    Serial.println();

    // stop
    Serial.println("Done!");
    delay(1000);
    //while(1){
    //}
}
//<----------------------------------------------------------
// Definition of ConvertUnixTimeToLocalTime function
// derived/forked from http://mrkk.ciao.jp/memorandom/unixtime/unixtime.html
#define ARRAYSIZE(_arr) (sizeof(_arr) / sizeof(_arr[0]))
#define TIME_OFFSET 9*60*60
#define SECONDS_IN_A_DAY    (24*60*60)
#define EPOCH_DAY       (1969*365L + 1969/4 - 1969/100 + 1969/400 + 306)    //  days from 0000/03/01 to 1970/01/01
#define UNIX_EPOCH_DAY  EPOCH_DAY
#define YEAR_ONE        365
#define YEAR_FOUR       (YEAR_ONE * 4 + 1)  //  it is YEAR_ONE*4+1 so the maximum reminder of day / YEAR_FOUR is YEAR_ONE * 4 and it occurs only on 2/29
#define YEAR_100        (YEAR_FOUR * 25 - 1)
#define YEAR_400        (YEAR_100*4 + 1)    //  it is YEAR_100*4+1 so the maximum reminder of day / YEAR_400 is YEAR_100 * 4 and it occurs only on 2/29

void ConvertUnixTimeToLocalTime(uint64_t unixtime, uint32_t *pyear, uint8_t *pmonth, uint8_t *pday, uint8_t *phour, uint8_t *pminute, uint8_t *psecond) {
    uint32_t unixday;
    uint16_t year = 0;
    uint8_t  leap = 0;
    uint32_t n;
    uint8_t month, day, weekday;
    uint8_t hour, minute, second;
    static const uint16_t monthday[] = { 0,31,61,92,122,153,184,214,245,275,306,337 };

    unixtime += TIME_OFFSET;
    second = unixtime % 60;
    minute = (unixtime / 60) % 60;
    hour = (unixtime / 3600) % 24;

    unixday = (uint32_t)(unixtime / SECONDS_IN_A_DAY);
    weekday = (uint8_t)((unixday + 3) % 7); //  because the unix epoch day is thursday
    unixday += UNIX_EPOCH_DAY;  //  days from 0000/03/01 to 1970/01/01

    year += 400 * (unixday / YEAR_400);
    unixday %= YEAR_400;

    n = unixday / YEAR_100;
    year += n * 100;
    unixday %= YEAR_100;

    if (n == 4){
        leap = 1;
    } else {
        year += 4 * (unixday / YEAR_FOUR);
        unixday %= YEAR_FOUR;
        
        n = unixday / YEAR_ONE;
        year += n;
        unixday %= YEAR_ONE;
        if (n == 4) {
            leap = 1;
        }
    }
    if (leap != 0) {
        month = 2;
        day = 29;
    }
    else {
        month = (unixday * 5 + 2) / 153;
        day = unixday - monthday[month] + 1;    //  
        month += 3;
        if (month > 12) {
            ++year;
            month -= 12;
        }
    }
    *psecond = second;
    *pminute = minute;
    *phour = hour;
    *pyear = year;
    *pmonth = month;
    *pday = day;
}
//>----------------------------------------------------------

```
以下の#defineでボードを切り換えているがボードの種類を設定すると、
システムで設定されている定義が有効になるので特にソースを変更する必要はない。
```

#define WIO_TERMINAL
#define ESP8266
#define ESP32
```

以下は、自分の環境に応じて変更する：
```

const char* ssid   = "your_ssid";
const char* passwd = "your_passwd";
```

書き込み後に「picocom /dev/ttyACM0 -b115200」または「picocom /dev/ttyUSB0 -b115200」で通信ソフトを起動すると以下のような出力が表示される：
```bash

$ picocom /dev/ttyACM0 -b115200

Terminal ready

--- Header ---
HTTP/1.1 200 OK
Connection: close
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: *
Access-Control-Expose-Headers: 
Cache-Control: max-age=0, private, must-revalidate
Content-Length: 345
Content-Type: application/json; charset=utf-8
Cross-Origin-Window-Policy: deny
Date: Mon, 03 Aug 2020 06:39:09 GMT
Server: Cowboy
X-Content-Type-Options: nosniff
X-Download-Options: noopen
X-Frame-Options: SAMEORIGIN
X-Permitted-Cross-Domain-Policies: none
X-Request-Id: c368a196-c927-43e2-b741-c3ab7ce11336
X-Runtime: 2ms
X-Xss-Protection: 1; mode=block
Via: 1.1 vegur

accessed BODY:
{"abbreviation":"JST","client_ip":"218.231.243.62","datetime":"2020-08-03T15:39:10.171382+09:00","day_of_week":1,"day_of_year":216,"dst":false,"dst_from":null,"dst_offset":0,"dst_until":null,"raw_offset":32400,"timezone":"Asia/Tokyo","unixtime":1596436750,"utc_datetime":"2020-08-03T06:39:10.171382+00:00","utc_offset":"+09:00","week_number":32}

Deserialized:
abbreviation:JST
client_ip:218.231.243.62
datetime:2020-08-03T15:39:10.171382+09:00
day_of_week:1
day_of_year:216
raw_offset:32400
timezone:Asia/Tokyo
unixtime:1596436750
utc_datetime:2020-08-03T06:39:10.171382+00:00
utc_offset:+09:00
week_number:32

Current Time from unixtime:
2020/08/03(Mon): 15:39:10

Done!
```
出力には、WorldTimeAPIで取得した時刻が出力される。

## 参考情報

[World Time API](http://worldtimeapi.org/)    
Simple JSON/plain-text API to obtain the current time in, and related data about, a timzone.  

[Wio-Terminal/ESP8622/ESP32ボードを共通のスケッチで動かす(HTTP-ACCESS編)](https://beta-notes.way-nifty.com/blog/2020/08/post-2f0b6e.html)  
[ESP8266 Arduino CoreでGETアクセス。HTTP/HTTPS＆プロクシを使ったHTTP通信](https://qiita.com/ichirowo/items/d794cee88ccd7f01ad7c)   
[ESP8266+ArduinoでHTTPS接続](https://www.motohasi.net/blog/?p=3885)   

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

以上
