
2020/7/7

PlatformIO cli Wio Terminal WiFi
# PlatformIO cli Wio Terminal WiFi

## 概要
Wio-TerminalでWiFiで使う(Seeeduino-Wio-Terminal/Arduino版)。  
開発ツールのインストールは「[開発ツールPlatformIOをcliで使う(Seeeduino-Wio-Terminal/Arduino版)](https://beta-notes.way-nifty.com/blog/2020/07/post-3f0abb.html)」を参照のこと。
(ホストPCとしてはubuntuを想定している)

## Wio-TerminalのWiFiファームウェアのアップデート
WiFi関係のスケッチを動かす前にWiFiのファームウェアをアップデートする：  
手順は以下を参照すること(現状、windows環境のみ)：   

[Wio TerminalをWi-Fiに繋ぐ](https://qiita.com/jksoft/items/cb11eb171002c0ed1f25)   
(ファームウェアアップデートについては本家サイトの説明よりも分かりやすい)  

## WiFi用デモ・スケッチ#1
WiFiネットワークからIPアドレスを取得するスケッチ：

src/wifi_getIPadrs.ino
```c++


#include <SPI.h> // forced
#include "AtWiFi.h"

const char* ssid = "your_ssid";
const char* password = "your_passwd";

void setup() {
    Serial.begin(115200);
    while(!Serial); // Wait for Serial to be ready
    delay(1000);

    // Set WiFi to station mode and disconnect from an AP if it was previously connected
    WiFi.mode(WIFI_STA);
    WiFi.disconnect();
    delay(2000);

    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.println("Connecting to WiFi..");
    }
    Serial.println("Connected to the WiFi network");
    Serial.print("IP Address: ");
    Serial.println (WiFi.localIP()); // prints out the device's IP address
    }

void loop() {

}
```

出力例：
```

picocom /dev/ttyACM0 -b115200

Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connected to the WiFi network
IP Address: 192.168.0.16

```

## WiFi用デモ・スケッチ#2
webにアクセスするスケッチ：

src/wifi_client.ino
```c++


#include <WiFiClientSecure.h>;

const char* ssid     = "your_ssid";
const char* password = "your_passwd";
 
const char*  server = "trends.google.co.jp";  // Server URL
  
// You can use x.509 client certificates if you want
//const char* test_client_key = "";   //to verify the client
//const char* test_client_cert = "";  //to verify the client
  
WiFiClientSecure client;
  
void setup() {
    //Initialize serial and wait for port to open:
    Serial.begin(115200);
    while(!Serial); // Wait for Serial to be ready
    delay(1000);
  
    Serial.print("Attempting to connect to SSID: ");
    //Serial.println(ssid);
    Serial.println("");
    WiFi.begin(ssid, password);
  
    // attempt to connect to Wifi network:
    while (WiFi.status() != WL_CONNECTED) {
        Serial.print(".");
        // wait 1 second for re-trying
        delay(1000);
    }
    Serial.print("Connected");
    //Serial.println(ssid);
    Serial.println("");
  
    //client.setCACert(test_root_ca);
    //client.setCertificate(test_client_key); // for client verification
    //client.setPrivateKey(test_client_cert); // for client verification
  
    Serial.println("\nStarting connection to server...");
    if (!client.connect(server, 443)) {
        Serial.println("Connection failed!");
    } else {
        Serial.println("Connected to server!");
        // Make a HTTP request:
        client.println("GET /trends/trendingsearches/daily/rss?geo=JP HTTP/1.0");
        client.println("Host: trends.google.co.jp");
        client.println("Connection: close");
        client.println();
  
        while (client.connected()) {
            String line = client.readStringUntil('\n');
            if (line == "\r") {
                Serial.println("headers received");
                break;
            }
        }
        // if there are incoming bytes available
        // from the server, read them and print them:
        while (client.available()) {
            char c = client.read();
            if (c == '\n') {
                Serial.write('\r');
            }
            Serial.write(c);
        }
        client.stop();
    }
}
  
void loop() {
    // do nothing
}
```

出力例：
```

picocom /dev/ttyACM0 -b115200

Attempting to connect to SSID: 
.......Connected

Starting connection to server...
Connected to server!
headers received
<?xml version='1.0' encoding='UTF-8' standalone='yes'?>
<rss xmlns:atom='http://www.w3.org/2005/Atom' xmlns:ht='https://trends.google.co.jp/trends/trendingsearches/daily' version='2.0'>
	<channel>
		<title>Daily Search Trends</title>
		<description>Recent searches</description>
		<link>https://trends.google.co.jp/trends/trendingsearches/daily?geo=JP</link>
		<atom:link href='https://trends.google.co.jp/trends/trendingsearches/daily/rss?geo=JP' rel='self' type='application/rss+xml'/>
		<item>
			<title>水樹奈々</title>
			<ht:approx_traffic>200,000+</ht:approx_traffic>
			<description>水樹奈々結婚</description>
			<link>https://trends.google.co.jp/trends/trendingsearches/daily?geo=JP#%E6%B0%B4%E6%A8%B9%E5%A5%88%E3%80%85</link>
			<pubDate>Tue, 07 Jul 2020 13:00:00 +0900</pubDate>
			<ht:picture>https://t3.gstatic.com/images?q=tbn:ANd9GcRLtm6WxBIKuZi_N2LNK6KfQ0O8KalM-H4xFcZyZ-jClbEzkOwSm94L34oDZfEZVmJNf1M0-8fP</ht:picture>
			<ht:picture_source>Yahoo!ニュース</ht:picture_source>
			<ht:news_item>
				<ht:news_item_title>水樹奈々、結婚発表 お相手は“音楽関係”「40歳、デビュー20周年という ...</ht:news_item_title>
				<ht:news_item_snippet>7月7日、声優・歌手として活動する水樹奈々が入籍したことを自身のブログで発表。お相手は「音楽関係の仕事をしている方」とのこと。</ht:news_item_snippet>

...
...
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
