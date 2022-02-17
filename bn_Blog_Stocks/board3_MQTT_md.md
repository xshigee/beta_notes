
2020/7/21:  
初版

board3 MQTT
# board3 MQTT

## 概要
Wio-Terminal/ESP8622/ESP32ボードを共通のスケッチで動かす(MQTT編)    
(ホストPCとしてはubuntuを想定している)


## 外部ライブラリ(platformioの場合)
以下を実行してライブラリをインストールする：
```bash

# MQTT lib インストール
pio lib install 617

```

## 外部ライブラリ(Arduino-IDEの場合)
Arduino-IDEでMQTTライブラリをインストールする。
MQTTは同じようなライブラリが見つかるが
「author:Joel Gaehwiler」のものをインストールする。

## デモ・スケッチ
  
src/MQTT_test.ino
```c++

// select board
////#define WIO_TERMINAL
////#define ESP8266
////#define ESP32
//------------------

// 
// MQTT program for Wio-Terminal/ESP8266/ESP32
//
// Forked from the following code:
//--------------------------------------------
// This example uses an Adafruit Huzzah ESP8266
// to connect to shiftr.io.
//
// You can check on your device after a successful
// connection here: https://shiftr.io/try.
//
// by Joël Gähwiler
// https://github.com/256dpi/arduino-mqtt
//--------------------------------------------

#ifdef WIO_TERMINAL
#include <AtWiFi.h>
#endif
#ifdef ESP8266
#include <ESP8266WiFi.h>
#endif
#ifdef ESP32
#include <WiFi.h>
#endif

#include <MQTT.h>

const char ssid[] = "your_ssid";
const char pass[] = "your_passwd";

WiFiClient net;
MQTTClient client;

unsigned long lastMillis = 0;

void connect() {
  Serial.print("checking wifi...");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(1000);
  }

  Serial.print("\nconnecting...");
  //while (!client.connect("arduino", "try", "try")) { // for "broker.shiftr.io"
  while (!client.connect("arduino", "", "")) { // no username. no passwd (for "mqtt.eclipse.org")
    Serial.print(".");
    delay(1000);
  }

  Serial.println("\nconnected!");

  client.subscribe("topic0");
  // client.unsubscribe("/hello");
}

void messageReceived(String &topic, String &payload) {
  Serial.println("incoming: " + topic + " - " + payload);
}

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, pass);

  // Note: Local domain names (e.g. "Computer.local" on OSX) are not supported by Arduino.
  // You need to set the IP address directly.

  // uncomment one of them to select MQTT server
  //client.begin("broker.shiftr.io", net); //OK
  client.begin("mqtt.eclipse.org", net); //OK
  //client.begin("test.mosquitto.org", net); //NG?

  client.onMessage(messageReceived);

  connect();
}

void loop() {
  char s[100]; // buf for sprintf

  // dumy sensor values
  float pascals = 101312;
  float altm = 42; //m
  float tempC = 28.7; // deg C

  client.loop();
  delay(10);  // <- fixes some issues with WiFi stability

  if (!client.connected()) {
    connect();
  }

  // publish a message roughly every second.
  if (millis() - lastMillis > 1000) {
    lastMillis = millis();

    // publish sensor values
    sprintf(s,"{ \"press\" : %.2f, \"altm\" : %.1f, \"tempC\" : %.1f}", 
            pascals/100, altm, tempC);
    client.publish("topic0", &s[0]);
  }
  delay(1000);
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
以下は利用するMQTTサーバー(broker)に合わせてコメントアウトする：
```

  //client.begin("broker.shiftr.io", net); //OK
  client.begin("mqtt.eclipse.org", net); //OK
```

書き込み後に「picocom /dev/ttyACM0 -b115200」または「picocom /dev/ttyUSB0 -b115200」で通信ソフトを起動すると以下のような出力が表示される：
```bash


$ picocom /dev/ttyACM0 -b115200

Terminal ready
# 実際のセンサーを接続していないので一定のダミーデータを受信する
incoming: topic0 - { "press" : 1013.12, "altm" : 42.0, "tempC" : 28.7}
incoming: topic0 - { "press" : 1013.12, "altm" : 42.0, "tempC" : 28.7}
incoming: topic0 - { "press" : 1013.12, "altm" : 42.0, "tempC" : 28.7}
incoming: topic0 - { "press" : 1013.12, "altm" : 42.0, "tempC" : 28.7}
incoming: topic0 - { "press" : 1013.12, "altm" : 42.0, "tempC" : 28.7}
incoming: topic0 - { "press" : 1013.12, "altm" : 42.0, "tempC" : 28.7}
...
<省略>

```

## 対向する送受信プログラム
以下をブラウザ(chromeのみ)で動かす：

MQTT_AT_webTestZero.html
```javascript

<html>

<script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>
<body>

<script>
//var mqtt_url = 'ws://test.mosquitto.org:8081'
var mqtt_url = 'ws://mqtt.eclipse.org/mqtt:443'
//var mqtt_url = 'wss://try:try@broker.shiftr.io'

document.body.innerHTML = '';
var consout = 'MQTT over WebSockets Test'+'('+mqtt_url+')<br>'
document.body.innerHTML = consout;

//var mqtt = require('mqtt');
var client = mqtt.connect(mqtt_url);

// subscribe Topic
//client.subscribe('hello');
client.subscribe('topic0');
//client.subscribe('topic0/sample/hello');
//client.subscribe('topic0/sample/#');
// wildcard topic
//client.subscribe('topic0/#');
//client.subscribe('#');

client.on('message', function(topic, payload) {
  console.log([topic, payload].join(': '));
  consout += [topic, payload].join(': ')+'<br>'
  document.body.innerHTML = consout
  // disconnect
  //client.end();
});

// publish messages
client.publish('topic0', 'hello world of MQTT! #1');
client.publish('topic0', 'hello world of MQTT! #2');
client.publish('topic0', 'hello world of MQTT! #3');
client.publish('topic0', 'hello world of MQTT! #4');
client.publish('topic0', 'hello world of MQTT! #5');

</script>

</body>

</html>
```

デモプログラムを起動後、上の「MQTT_AT_webTestZero.html」をプラウザーで起動すると  
以下のweb画面が表示される(例)：
```
MQTT over WebSockets Test(ws://mqtt.eclipse.org/mqtt:443)
topic0: hello world of MQTT! #1
topic0: hello world of MQTT! #2
topic0: hello world of MQTT! #3
topic0: hello world of MQTT! #4
topic0: hello world of MQTT! #5
topic0: { "press" : 1013.12, "altm" : 42.0, "tempC" : 28.7}
topic0: { "press" : 1013.12, "altm" : 42.0, "tempC" : 28.7}
topic0: { "press" : 1013.12, "altm" : 42.0, "tempC" : 28.7}
topic0: { "press" : 1013.12, "altm" : 42.0, "tempC" : 28.7}
topic0: { "press" : 1013.12, "altm" : 42.0, "tempC" : 28.7}
topic0: { "press" : 1013.12, "altm" : 42.0, "tempC" : 28.7}
...

```

## Arduino-IDEのボードマネージャーのURL
以下のURLを設定することで３つのボード(Wio-Terminal/ESP8266/ESP32)が使用可能になる。
```

https://files.seeedstudio.com/arduino/package_seeeduino_boards_index.json
http://arduino.esp8266.com/stable/package_esp8266com_index.json
https://dl.espressif.com/dl/package_esp32_index.json
```


## 参考情報

https://docs.shiftr.io/interfaces/mqtt/
```
The following ports are available on the broker.shiftr.io host:

MQTT	MQTTS	MQTTWS	MQTTWSS
1883	8883	80	443

```

MQTT test server @mosquitto.org
```
https://test.mosquitto.org/
The server listens on the following ports:
1883 : MQTT, unencrypted
8883 : MQTT, encrypted
8884 : MQTT, encrypted, client certificate required
8080 : MQTT over WebSockets, unencrypted
8081 : MQTT over WebSockets, encrypted
websocketで使用するurlは以下になる：
'ws://test.mosquitto.org:8081'
```

MQTT test server @eclipse.org
```
http://mqtt.eclipse.org/
This is a public test MQTT broker service.
It currently listens on the following ports:
1883 : MQTT over unencrypted TCP
8883 : MQTT over encrypted TCP
80 : MQTT over unencrypted WebSockets (note: URL must be /mqtt )
443 : MQTT over encrypted WebSockets (note: URL must be /mqtt )
websocketで使用するurlは以下になる：
'ws://mqtt.eclipse.org/mqtt:443'
```

[10 Free Public & Private MQTT Brokers(For Testing & Production)](https://mntolia.com/10-free-public-private-mqtt-brokers-for-testing-prototyping/)

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

[ESP-WROOM-02ボードでMQTTとMPL3115A2(気圧・高度・気温センサ)を動かす（Arduino版）](https://beta-notes.way-nifty.com/blog/2020/07/post-db9550.html)   
本記事のスケッチは、この記事のスケッチとほとんど同じであるので参考にできる。  


以上
