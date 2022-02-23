
2020/7/25:  
初版

board3 REST API 2
# board3 REST API 2

## 概要
Wio-Terminal/ESP8622/ESP32ボードを共通のスケッチで動かす(REST-API2編)    
「[Wio-Terminal/ESP8622/ESP32ボードを共通のスケッチで動かす(REST-API編)](https://beta-notes.way-nifty.com/blog/2020/07/post-1da2d9.html)」で、Wio-Terminalのスケッチが動作しなかったので、その代わりのスケッチを提供する。
(ホストPCとしてはubuntuを想定している)


## 外部ライブラリ(platformioの場合)
以下を実行してライブラリをインストールする：
```bash

# mDNS lib インストール
#pio lib install "ArduinoMDNS"
pio lib install 2848

```

## デモ・スケッチ
  
src/REST-API2_test.ino
```c++

// select board
////#define WIO_TERMINAL
////#define ESP8266
////#define ESP32
//------------------

// REST-API server2 for Wio-Terminal/ESP8266/ESP32
/*
    Rest-API server

    Forked from arduino-esp32 on 04 July, 2018
    by Elochukwu Ifediora (fedy0)
*/
 
#ifdef WIO_TERMINAL
#include <AtWiFi.h>
#include <WiFiUdp.h>
#include <ArduinoMDNS.h>
#endif
#ifdef ESP8266
#include <ESP8266WiFi.h>
#include <ESP8266mDNS.h>
#endif
#ifdef ESP32
#include <WiFi.h>
#include <ESPmDNS.h>
#endif
 
//#define LED 2   // Set the GPIO pin where you connected your test LED
 
// Change to your setting
const char* ssid = "your_ssid";
const char* password = "your_passwd";

#ifdef WIO_TERMINAL
WiFiUDP udp;
MDNS mdns(udp);
#endif

WiFiServer server(80);

void setup() {
    //set LED of your board
    //pinMode(LED, OUTPUT);
 
    Serial.begin(115200);
    while(!Serial); // Wait for Serial to be ready
    delay(1000);
    Serial.println();
    Serial.println("Configuring access point...");
 
    // set your SoftAP if you need
    //WiFi.softAP(ssid, password);
    //IPAddress myIP = WiFi.softAPIP();
    //or
    // set for station mode
    WiFi.mode(WIFI_STA);
    WiFi.begin(ssid, password);
    // Wait for connection
    while (WiFi.status() != WL_CONNECTED) {
      delay(500);
      Serial.print(".");
    } 
    IPAddress myIP = WiFi.localIP();
    
    Serial.println();
    Serial.print("AP IP address: ");
    Serial.println(myIP);
    server.begin();
 
    Serial.println("Server started");

  // Activate mDNS this is used to be able to connect to the server
  // with local DNS hostmane esp8266.local etc
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
}
 
void loop() {
    WiFiClient client = server.available();   // listen for incoming clients
 
    if (client) {  // if you get a client,
        Serial.println("New Client.");  // print a message out the serial port
        String currentLine = "";  // make a String to hold incoming data from the client
        while (client.connected()) {   // loop while the client's connected
            if (client.available()) {  // if there's bytes to read from the client,
                char c = client.read();  // read a byte, then
                /////Serial.write(c);    // print it out the serial monitor
                if (c == '\n') {  // if the byte is a newline character
                    // if the current line is blank, you got two newline characters in a row.
                    // that's the end of the client HTTP request, so send a response:
                    if (currentLine.length() == 0) {
                        // HTTP headers always start with a response code (e.g. HTTP/1.1 200 OK)
                        // and a content-type so the client knows what's coming, then a blank line:
                        ////client.println("HTTP/1.1 200 OK");
                        ////client.println("Content-type:text/html");
                        ////client.println();
 
                        // the content of the HTTP response follows the header:
                        ////client.print("Click <a href=\"/H\">here</a> to turn ON the LED.<br>");
                        ////client.print("Click <a href=\"/L\">here</a> to turn OFF the LED.<br>");
 
                        // The HTTP response ends with another blank line:
                        client.println();
                        // break out of the while loop:
                        break;
                    } else {    // if you got a newline, then clear currentLine:
                        currentLine = "";
                    }
                } else if (c != '\r') {  // if you got anything else but a carriage return character,
                    currentLine += c;      // add it to the end of the currentLine
                }
 
                // Check to see if the client request was "GET /H" or "GET /L":
                //if (currentLine.endsWith("GET /H")) {
                if (currentLine.endsWith("/H")) {
                    //digitalWrite(LED, HIGH);               // GET /H turns the LED on
                    Serial.println("****ON****");
                    currentLine ="";
                    client.println("{ \"LED\" : \"on\"  }");
                }
                //if (currentLine.endsWith("GET /L")) {
                if (currentLine.endsWith("/L")) {
                    //digitalWrite(LED, LOW);                // GET /L turns the LED off
                    Serial.println("****OFF****");
                    currentLine ="";
                    client.println("{ \"LED\" : \"off\"  }");
                }

                if (currentLine.endsWith("/api/v1/led=1")) {
                    //digitalWrite(LED, HIGH);
                    Serial.println("led: ON");
                    currentLine ="";
                    client.println("{ \"LED\" : \"on\"  }");
                }

                if (currentLine.endsWith("/api/v1/led=0")) {
                    //digitalWrite(LED, LOW);
                    Serial.println("led: OFF");
                    currentLine ="";
                    client.println("{ \"LED\" : \"off\"  }");
                }

            }
        }
        // close the connection:
        client.stop();
        Serial.println("Client Disconnected.");
    }

#ifdef WIO_TERMINAL
  mdns.run(); // This actually runs the mDNS module. YOU HAVE TO CALL THIS PERIODICALLY
#endif
#ifdef ESP8266
  MDNS.update();
#endif
#ifdef ESP32
  //MDNS.update(); // no need to update on ESP32
#endif

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
Configuring access point...
............
AP IP address: 192.168.0.5
Server started
MDNS responder started(esp8266)

# ESP32の場合
Configuring access point...
..
AP IP address: 192.168.0.15
Server started
MDNS responder started(esp32)

# Wio-Terminalの場合
Configuring access point...
..............
AP IP address: 192.168.0.29
Server started
MDNS responder started(wiot)
```


curlによるコマンド実行例：
```

# ESP8266の場合
$ curl 'http://esp8266.local:80/api/v1/led=1'
{"LED": "on"}
$ curl 'http://esp8266.local:80/api/v1/led=0'
{"LED": "off"}

# ESP32の場合
$ curl 'http://esp32.local:80/api/v1/led=0'
{"LED": "off"}
$ curl 'http://esp32.local:80/api/v1/led=1'
{"LED": "on"}

#Wio-Terminalの場合
$ curl 'http://192.168.0.29:80/api/v1/led=0'
{ "LED" : "off"  }
$ curl 'http://192.168.0.29:80/api/v1/led=1'
{ "LED" : "on"  }
$ curl 'http://wiot.local:80/api/v1/led=0'
curl: (6) Could not resolve host: wiot.local
```

## 不具合
mDNSが動作していないので、wiot.localではなく、IPアドレスでアクセスすることになる。(原因不明)

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
