
2020/7/24:  
初版

board3 REST API
# board3 REST API

## 概要
Wio-Terminal/ESP8622/ESP32ボードを共通のスケッチで動かす(REST-API編)    
(ホストPCとしてはubuntuを想定している)


## 外部ライブラリ(platformioの場合)
以下を実行してライブラリをインストールする：
```bash

# mDNS lib インストール
#pio lib install "ArduinoMDNS"
pio lib install 2848

```

## デモ・スケッチ
  
src/REST-API_test.ino
```c++

// select board
////#define WIO_TERMINAL
////#define ESP8266
////#define ESP32
//------------------

// REST-API server for Wio-Terminal/ESP8266/ESP32
/*
 * Forked from the following code:
 * 
 *  Simple hello world Json REST response
 *  by Mischianti Renzo <https://www.mischianti.org>
 *
 *  https://www.mischianti.org/
 *
 */
#ifdef WIO_TERMINAL
#include <AtWiFi.h>
#include <WebServer.h>
#include <WiFiUdp.h>
#include <ArduinoMDNS.h>
#endif
#ifdef ESP8266
#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>
#include <ESP8266mDNS.h>
#endif
#ifdef ESP32
#include <WiFi.h>
#include <WebServer.h>
#include <ESPmDNS.h>
#endif

const char* ssid = "<your_ssid>";
const char* password = "<your_passwd>";

#ifdef WIO_TERMINAL
WiFiUDP udp;
MDNS mdns(udp);
WebServer server(80);
#endif
#ifdef ESP8266 
ESP8266WebServer server(80);
#endif
#ifdef ESP32
WebServer server(80);
#endif

// Serving Hello world
void getHelloWord() {
    server.send(200, "text/json", "{\"name\": \"Hello world\"}\r\n");
}
void ledOff() {
    // do something for LED off
    server.send(200, "text/json", "{\"led\": \"OFF\"}\r\n");
}
void ledOn() {
    // do something for LED on
    server.send(200, "text/json", "{\"led\": \"ON\"}\r\n");
}
 
// Define routing
void restServerRouting() {
    server.on("/", HTTP_GET, []() {
        server.send(200, F("text/html"),
            F("Welcome to the REST Web Server"));
    });
    server.on(F("/helloWorld"), HTTP_GET, getHelloWord);
    server.on(F("/api/v1/led=0"), HTTP_GET, ledOff);
    server.on(F("/api/v1/led=1"), HTTP_GET, ledOn);
}
 
// Manage not found URL
void handleNotFound() {
  String message = "File Not Found\n\n";
  message += "URI: ";
  message += server.uri();
  message += "\nMethod: ";
  message += (server.method() == HTTP_GET) ? "GET" : "POST";
  message += "\nArguments: ";
  message += server.args();
  message += "\n";
  for (uint8_t i = 0; i < server.args(); i++) {
    message += " " + server.argName(i) + ": " + server.arg(i) + "\n";
  }
  server.send(404, "text/plain", message);
}
 
void setup(void) {
  Serial.begin(115200);
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  Serial.println("");
 
  // Wait for connection
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("Connected to ");
  Serial.println(ssid);
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
 
  // Activate mDNS this is used to be able to connect to the server
  // with local DNS hostmane esp8266.local
#ifdef WIO_TERMINAL
  mdns.begin(WiFi.localIP(), "wiot");
  Serial.println("MDNS responder started(wiot)");
#endif
#ifdef ESP8266
  if (MDNS.begin("esp8266")) {
    Serial.println("MDNS responder started(esp8266)");
  }
#endif
#ifdef ESP32
  if (MDNS.begin("esp32")) {
    Serial.println("MDNS responder started(esp32)");
  }
#endif
 
  // Set server routing
  restServerRouting();
  // Set not found response
  server.onNotFound(handleNotFound);
  // Start server
  server.begin();
  Serial.println("HTTP server started");
}
 
void loop(void) {

#ifdef WIO_TERMINAL
  mdns.run(); // This actually runs the mDNS module. YOU HAVE TO CALL THIS PERIODICALLY
#endif
#ifdef ESP8266
  MDNS.update();
#endif
#ifdef ESP32
  //MDNS.update(); // no need to update on ESP32
#endif

  server.handleClient();
}

 ```

以下の#defineでボードを切り換えているがボードの種類を設定すると、
システムで設定されている定義が有効になるので特にソースを変更する必要はない。
```

#define WIO_TERMINAL
#define ESP8266
#define ESP32
```

以下については、自分の環境に合わせて変更すること：  
```

const char ssid[] = "yours_ssid";
const char pass[] = "yours_passwd";
```


書き込み後に「picocom /dev/ttyACM0 -b115200」または「picocom /dev/ttyUSB0 -b115200」で通信ソフトを起動すると以下のような出力が表示される：
```bash


$ picocom /dev/ttyACM0 -b115200

# ESP8266の場合
...
Connected to xxxxxxxx
IP address: 192.168.0.5
MDNS responder started(esp8266)
HTTP server started

# ESP32の場合
..
Connected to xxxxxxxx
IP address: 192.168.0.15
MDNS responder started(esp32)
HTTP server started

# Wio-Terminalの場合
Terminal ready
No ACK, R00
Device reset detected!
ESP-AT Lib initialized!

..................
Connected to xxxxxxxx
IP address: 192.168.0.29
MDNS responder started(wiot)
HTTP server started

```


curlによるコマンド実行例：
```

# ESP8266の場合
$ curl 'http://esp8266.local:80/helloWorld'
{"name": "Hello world"}
$ curl 'http://esp8266.local:80/api/v1/led=1'
{"led": "ON"}
$ curl 'http://esp8266.local:80/api/v1/led=0'
{"led": "OFF"}

# ESP32の場合
$ curl 'http://esp32.local:80/helloWorld'
{"name": "Hello world"}
$ curl 'http://esp32.local:80/api/v1/led=0'
{"led": "OFF"}
$ curl 'http://esp32.local:80/api/v1/led=1'
{"led": "ON"}
```

## 不具合
Wio-Terminalの場合、REST-APIは、起動後の１回しか動作しなかった。（原因不明）
mDNSも動作していないので、wiot.localではなく、IPアドレスでアクセスすることになる。


## Arduino-IDEのボードマネージャーのURL
以下のURLを設定することで３つのボード(Wio-Terminal/ESP8266/ESP32)が使用可能になる。
```

https://files.seeedstudio.com/arduino/package_seeeduino_boards_index.json
http://arduino.esp8266.com/stable/package_esp8266com_index.json
https://dl.espressif.com/dl/package_esp32_index.json
```
## 参考情報

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

以上
