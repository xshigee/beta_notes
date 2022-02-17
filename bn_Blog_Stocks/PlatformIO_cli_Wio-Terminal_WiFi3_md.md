
2020/7/11

PlatformIO cli Wio Terminal WiFi3
# PlatformIO cli Wio Terminal WiFi3

## 概要
Wio-TerminalでWiFiで使う(その３: REST-API)      
本記事は、[[Wio-TerminalでWiFiで使う(その２: STARTWARS,WEB_ACCESS)](https://beta-notes.way-nifty.com/blog/2020/07/post-ea61f5.html)]の続きになる。   
(ホストPCとしてはubuntuを想定している)


## wifi_rest_api
REST-APIのwebサーバーのスケッチ：

src/wifi_rest_api.ino
```c++


/*
    Rest-API server

    Forked from arduino-esp32 on 04 July, 2018
    by Elochukwu Ifediora (fedy0)
*/
 
#include <AtWiFi.h>
#include <WiFiClient.h>
#include <WiFiAP.h>
 
#define LED 2   // Set the GPIO pin where you connected your test LED
 
// Set these to your desired credentials.
const char* ssid = "MyAP";
const char* password = "mypasswd";
 
WiFiServer server(80);

void setup() {
    pinMode(LED, OUTPUT);
 
    Serial.begin(115200);
    while(!Serial); // Wait for Serial to be ready
    delay(1000);
    Serial.println();
    Serial.println("Configuring access point...");
 
    // You can remove the password parameter if you want the AP to be open.
    WiFi.softAP(ssid, password);
    IPAddress myIP = WiFi.softAPIP();
    /*
     * Note 1
     * Record this IP, will used by Client (such as Web Browser)
     */
    Serial.print("AP IP address: ");
    Serial.println(myIP);
    server.begin();
 
    Serial.println("Server started");
}
 
void loop() {
    WiFiClient client = server.available();   // listen for incoming clients
 
    if (client) {                             // if you get a client,
        Serial.println("New Client.");           // print a message out the serial port
        String currentLine = "";                // make a String to hold incoming data from the client
        while (client.connected()) {            // loop while the client's connected
            if (client.available()) {             // if there's bytes to read from the client,
                char c = client.read();             // read a byte, then
                /////Serial.write(c);                    // print it out the serial monitor
                if (c == '\n') {                    // if the byte is a newline character
 
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
                    digitalWrite(LED, HIGH);               // GET /H turns the LED on
                    Serial.println("****ON****");
                    currentLine ="";
                    client.println("{ \"LED\" : \"on\"  }");
                }
                //if (currentLine.endsWith("GET /L")) {
                if (currentLine.endsWith("/L")) {
                    digitalWrite(LED, LOW);                // GET /L turns the LED off
                    Serial.println("****OFF****");
                    currentLine ="";
                    client.println("{ \"LED\" : \"off\"  }");
                }

                if (currentLine.endsWith("/api/v1/led=1")) {
                    digitalWrite(LED, HIGH);
                    Serial.println("led: ON");
                    currentLine ="";
                    client.println("{ \"LED\" : \"on\"  }");
                }

                if (currentLine.endsWith("/api/v1/led=0")) {
                    digitalWrite(LED, LOW);
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
}
```
スケッチ実行後、SSIDとして「MyAP」が見えるので
パスワードとして"mypasswd"を入力してWiFi接続する。
その後、以下のようなコマンドでwio-terminalを制御できる：
(curlによる)コマンド実行例：   
```

$ curl 'http://192.168.43.1:80/api/v1/led=0'
{ "LED" : "off"  }

$ curl 'http://192.168.43.1:80/api/v1/led=1'
{ "LED" : "on"  }

$ curl 'http://192.168.43.1:80/api/v1/led=0'
{ "LED" : "off"  }

$ curl 'http://192.168.43.1:80/api/v1/led=1'
{ "LED" : "on"  }

$ curl 'http://192.168.43.1:80/L'
{ "LED" : "off"  }

$ curl 'http://192.168.43.1:80/H'
{ "LED" : "on"  }

```

出力例：
```

picocom /dev/ttyACM0 -b115200

Configuring access point...
AP IP address: 192.168.43.1
Server started
New Client.
led: OFF
Client Disconnected.
New Client.
led: ON
Client Disconnected.
New Client.
led: OFF
Client Disconnected.
New Client.
led: ON
Client Disconnected.
```


## 参考情報

[Platform/Wio Terminal Network/Wi-Fi - example sketches](https://wiki.seeedstudio.com/Wio-Terminal-Wi-Fi/)  

[Wio Terminalをはじめよう(ピン配置、データシート、回路図など)](https://wiki.seeedstudio.com/jp/Wio-Terminal-Getting-Started/)  

[Platform/Wio Terminal/Network/Overview](https://wiki.seeedstudio.com/Wio-Terminal-Network-Overview/)  
download url:   
https://files.seeedstudio.com/wiki/Wio-Terminal/res/ameba-image-Tool-v2.4.1.zip  
https://files.seeedstudio.com/wiki/Wio-Terminal/res/20200601-rtl8720d-images-v2.2.0.0.zip  

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

以上
