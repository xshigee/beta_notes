
2020/8/2:  
初版

board3 HTTP ACCESS
# board3 HTTP ACCESS

## 概要
Wio-Terminal/ESP8622/ESP32ボードを共通のスケッチで動かす(HTTP-ACCESS編)    
(ホストPCとしてはubuntuを想定している)

## デモ・スケッチ  
src/HTTP-ACCESS.ino
```c++

// select board
////#define WIO_TERMINAL
////#define ESP8266
////#define ESP32
//------------------

// 
// HTTP access for Wio-Terminal/ESP8266/ESP32
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

#define BODY_SIZE 1024
char bodyBuff[BODY_SIZE];

const char* ssid   = "your_ssid";
const char* passwd = "your_passwd";

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

    // stop
    Serial.println("Done!");
    while(1){
    }
}
 ```
スケッチのテスト用urlなどは[World Time API](http://worldtimeapi.org/)のものを使用している。
ここも必要に応じて変更できる。

以下の#defineでボードを切り換えているがボードの種類を設定すると、
システムで設定されている定義が有効になるので特にソースを変更する必要はない。
```

#define WIO_TERMINAL
#define ESP8266
#define ESP32
```

以下については、自分の環境に合わせて変更すること：  
```

const char* ssid   = "your_ssid";
const char* passwd = "your_passwd";
```

書き込み後に「picocom /dev/ttyACM0 -b115200」または「picocom /dev/ttyUSB0 -b115200」で通信ソフトを起動すると以下のような出力が表示される：
```bash


$ picocom /dev/ttyACM0 -b115200

Terminal ready
Connecting to xxxxxxxxxxxx
...............
WiFi connected
IP address: 
192.168.0.4

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
Date: Sun, 02 Aug 2020 14:42:08 GMT
Server: Cowboy
X-Content-Type-Options: nosniff
X-Download-Options: noopen
X-Frame-Options: SAMEORIGIN
X-Permitted-Cross-Domain-Policies: none
X-Request-Id: 0813b454-28fa-44f7-a705-76c3b42eb739
X-Runtime: 1ms
X-Xss-Protection: 1; mode=block
Via: 1.1 vegur

accessed BODY:
{"abbreviation":"JST","client_ip":"218.231.243.62","datetime":"2020-08-02T23:42:09.116136+09:00","day_of_week":0,"day_of_year":215,"dst":false,"dst_from":null,"dst_offset":0,"dst_until":null,"raw_offset":32400,"timezone":"Asia/Tokyo","unixtime":1596379329,"utc_datetime":"2020-08-02T14:42:09.116136+00:00","utc_offset":"+09:00","week_number":31}
Done!
```

## Arduino-IDEのボードマネージャーのURL
以下のURLを設定することで３つのボード(Wio-Terminal/ESP8266/ESP32)が使用可能になる。
```

https://files.seeedstudio.com/arduino/package_seeeduino_boards_index.json
http://arduino.esp8266.com/stable/package_esp8266com_index.json
https://dl.espressif.com/dl/package_esp32_index.json
```

## 参考情報

[World Time API](http://worldtimeapi.org/)    
Simple JSON/plain-text API to obtain the current time in, and related data about, a timzone.  

[ESP8266 Arduino CoreでGETアクセス。HTTP/HTTPS＆プロクシを使ったHTTP通信](https://qiita.com/ichirowo/items/d794cee88ccd7f01ad7c)   
[ESP8266+ArduinoでHTTPS接続](https://www.motohasi.net/blog/?p=3885)   


[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

以上
