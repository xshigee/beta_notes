
2020/7/5:  
初版

PlatformIO ESP-WROOM-02 MQTT MPL3115A2
# PlatformIO ESP-WROOM-02 MQTT MPL3115A2

## 概要
以下のESP-WROOM-02ボードでMQTTと以下のMPL3115A2(気圧・高度・気温センサ)を動かす（Arduino版）。
(ホストPCとしてはubuntuを想定している)

[ESPr® Developer（ESP-WROOM-02開発ボード）](https://www.switch-science.com/catalog/2500/)   
[MPL3115A2 - I2C気圧・高度・気温センサ](https://www.switch-science.com/catalog/1780/)  

## PINOUT   
部品面から見てボードのUSBコネクタを下にした配置

|ESP-WROOM-02(Right)|
| :--- |
|GND|
|IO16|
|TOUT|
|RESET|
|IO5|
|TXD|
|RXD|
|IO4|
|GND|
|VOUT|

| ESP-WROOM-02(Left) |
| :--- |
|3V3|
|EN|
|IO1|
|IO12|
|IO13|
|IO15|
|IO2|
|IO0|
|GND|
|VIN|

## 接続
| ESP-WROOM-02 | MPL3115A2 |
| ---: | :--- |
| IO5 | SCL |
| IO4 | SDA |
| GND | GND |
| 3V3 | Vin |

## platformio.ini
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
```

## 外部ライブラリ
以下を実行してライブラリをインストールする：
```bash

# MQTT lib インストール
pio lib install 617

# Adafruit MPL3115A2 Lib インストール
pio lib install 796

#実際に、これだと、コンパイルがエラーになったので、
#以下のように直接ライブラリのソースをダウンロードする：

cd src
wget https://raw.githubusercontent.com/adafruit/Adafruit_MPL3115A2_Library/master/Adafruit_MPL3115A2.cpp
wget https://raw.githubusercontent.com/adafruit/Adafruit_MPL3115A2_Library/master/Adafruit_MPL3115A2.h

```

## MQTT_MPL3115A2_test
  
src/MQTT_MPL3115A2_test.ino
```c++

//
// MPL3115A2 sensor data publisher/subscriber
//
// MPL3115A2 Library url:
// https://github.com/adafruit/Adafruit_MPL3115A2_Library
// MQTT lib url:
// https://github.com/256dpi/arduino-mqtt

#include <Wire.h>
#include <Adafruit_MPL3115A2.h>
#include <ESP8266WiFi.h>
#include <MQTT.h>

const char ssid[] = "yours_ssid";
const char pass[] = "yours_passwd";

Adafruit_MPL3115A2 baro = Adafruit_MPL3115A2();

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
  while (!client.connect("", "", "")) { // no username. no passwd (for "mqtt.eclipse.org")
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
  //client.begin("broker.shiftr.io", net); //OK
  client.begin("mqtt.eclipse.org", net); //OK
  //client.begin("test.mosquitto.org", net); //NG?
  client.onMessage(messageReceived);

  connect();
}

void loop() {
  char s[100]; // buf for sprintf
  if (! baro.begin()) {
    Serial.println("Couldnt find sensor");
    return;
  }
  float pascals = baro.getPressure();
  Serial.print(pascals/100); Serial.println(" hPa");
  //
  float altm = baro.getAltitude();
  Serial.print(altm); Serial.println(" meters");
  //
  float tempC = baro.getTemperature();

  client.loop();
  delay(10);  // <- fixes some issues with WiFi stability

  if (!client.connected()) {
    connect();
  }

  // publish a message roughly every second.
  if (millis() - lastMillis > 1000) {
    lastMillis = millis();
    //client.publish("topic0", String((int)pascals));
    sprintf(s,"{ \"press\" : %.2f, \"altm\" : %.1f, \"tempC\" : %.1f}", pascals/100, altm, tempC);
    client.publish("topic0", &s[0]);
  }
  delay(2000);
}
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

書き込み後に「picocom /dev/ttyUSB0 -b115200」で通信ソフトを起動すると以下のような出力が表示される：
```bash


$ picocom /dev/ttyUSB0 -b115200

Terminal ready

1002.88 hPa
86.88 meters
incoming: topic0 - { "press" : 1002.88, "altm" : 87.000000.1, "tempC" : 28.1}
1002.86 hPa
87.00 meters
incoming: topic0 - { "press" : 1002.88, "altm" : 86.875000.1, "tempC" : 28.1}
1002.86 hPa
86.44 meters
incoming: topic0 - { "press" : 1002.86, "altm" : 87.000000.1, "tempC" : 28.1}
1002.86 hPa
86.50 meters
incoming: topic0 - { "press" : 1002.86, "altm" : 86.437500.1, "tempC" : 28.2}
1002.87 hPa
86.94 meters
incoming: topic0 - { "press" : 1002.86, "altm" : 86.500000.1, "tempC" : 28.2}
1002.90 hPa
87.06 meters
incoming: topic0 - { "press" : 1002.87, "altm" : 86.937500.1, "tempC" : 28.1}
1002.85 hPa
86.75 meters
incoming: topic0 - { "press" : 1002.90, "altm" : 87.062500.1, "tempC" : 28.1}
1002.89 hPa
86.56 meters
incoming: topic0 - { "press" : 1002.85, "altm" : 86.750000.1, "tempC" : 28.2}
1002.90 hPa
86.44 meters
incoming: topic0 - { "press" : 1002.89, "altm" : 86.562500.1, "tempC" : 28.2}
1002.90 hPa
86.69 meters
incoming: topic0 - { "press" : 1002.90, "altm" : 86.437500.1, "tempC" : 28.2}
1002.96 hPa
86.44 meters

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
topic0: { "press" : 1002.58, "altm" : 88.3, "tempC" : 28.7}
topic0: { "press" : 1002.58, "altm" : 88.4, "tempC" : 28.7}
topic0: { "press" : 1002.56, "altm" : 88.1, "tempC" : 28.8}
topic0: { "press" : 1002.61, "altm" : 88.3, "tempC" : 28.8}
topic0: { "press" : 1002.59, "altm" : 88.4, "tempC" : 28.7}
topic0: { "press" : 1002.59, "altm" : 88.4, "tempC" : 28.7}
topic0: { "press" : 1002.57, "altm" : 88.6, "tempC" : 28.7}
topic0: { "press" : 1002.59, "altm" : 89.0, "tempC" : 28.7}
topic0: { "press" : 1002.58, "altm" : 88.2, "tempC" : 28.7}
topic0: { "press" : 1002.61, "altm" : 88.2, "tempC" : 28.7}
topic0: { "press" : 1002.60, "altm" : 88.8, "tempC" : 28.7}
topic0: { "press" : 1002.59, "altm" : 88.6, "tempC" : 28.7}
topic0: { "press" : 1002.63, "altm" : 88.3, "tempC" : 28.7}
topic0: { "press" : 1002.59, "altm" : 88.3, "tempC" : 28.8}
topic0: { "press" : 1002.60, "altm" : 88.3, "tempC" : 28.7}
...

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

以上
