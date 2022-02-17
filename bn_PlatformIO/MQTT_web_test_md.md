
2020/3/4

MQTT web test
# MQTT web test

## 概要
MQTT(over WebSocket)をブラウザーで使ってみた。

## HTMLスクリプト

MQTTwebTest.html
```javascript

<html>

<script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>
<body>

<script>
document.body.innerHTML = '';
var consout = 'MQTT over WebSockets Test'+'<br>'
document.body.innerHTML = consout;

//var mqtt = require('mqtt')
var client = mqtt.connect('ws://test.mosquitto.org:8081')

// subscribe Topic
client.subscribe('mqtt/test');

client.on('message', function(topic, payload) {
  console.log([topic, payload].join(': '));
  consout += [topic, payload].join(': ')+'<br>'
  document.body.innerHTML = consout
  // disconnect
  //client.end();
});

// publish messages
client.publish('mqtt/test', 'hello world of MQTT! #1');
client.publish('mqtt/test', 'hello world of MQTT! #2');
client.publish('mqtt/test', 'hello world of MQTT! #3');
client.publish('mqtt/test', 'hello world of MQTT! #4');
client.publish('mqtt/test', 'hello world of MQTT! #5');

</script>

</body>

</html>
```
Brokerサーバーとしては、test.mosquitto.orgを使用した。(このサーバーは、「MQTT」と「MQTT over WebSocket」をサポートしている)

サーバーの状況によっては、レスポンスが悪い時があるので、辛抱強く？待つ必要があるようだ。

　　
このHTMLファイルをブラウザーで開くことで実行する。
(原因は不明だが、firefoxでは動作しないようだ。chromeでは動作した)

## 出力結果(ウェブ画面)

```

MQTT over WebSockets Test
mqtt/test: hello world of MQTT! #1
mqtt/test: hello world of MQTT! #2
mqtt/test: hello world of MQTT! #3
mqtt/test: hello world of MQTT! #4
mqtt/test: hello world of MQTT! #5
```
Topic「mqtt/test」に対するメッセージ「hello world of MQTT! #N」を受信している。

## おまけ(nodeでの使用例)

Installation:  

```bash

npm install mqtt --save
```

Example:  

hello_mqtt.js
```javascript

var mqtt    = require('mqtt');
var client  = mqtt.connect('mqtt://test.mosquitto.org');
//var client  = mqtt.connect('ws://test.mosquitto.org');

client.subscribe('presence');
client.publish('presence', 'Hello mqtt');

client.on('message', function (topic, message) {
  // message is Buffer
  console.log(message.toString());
});

client.end();

```
こちらのほうは、(over WebSocket ではなく)MQTTで接続する。


## 参考情報

[MQTT.js](https://github.com/mqttjs/MQTT.js/blob/master/README.md)  

[MQTT test server @mosquitto.org](https://test.mosquitto.org/)  
The server listens on the following ports:  
1883 : MQTT, unencrypted  
8883 : MQTT, encrypted  
8884 : MQTT, encrypted, client certificate required  
8080 : MQTT over WebSockets, unencrypted  
8081 : MQTT over WebSockets, encrypted  

[MQTT test server @eclipse.org](http://mqtt.eclipse.org/)  
This is a public test MQTT broker service.  
It currently listens on the following ports:  
1883 : MQTT over unencrypted TCP  
8883 : MQTT over encrypted TCP  
80 : MQTT over unencrypted WebSockets (note: URL must be /mqtt )  
443 : MQTT over encrypted WebSockets (note: URL must be /mqtt ) 
 
[10 Free Public & Private MQTT Brokers(For Testing & Production)](https://mntolia.com/10-free-public-private-mqtt-brokers-for-testing-prototyping/)  
[http://www.mqtt-dashboard.com/](http://www.mqtt-dashboard.com/)  

[ESP32/ESP8266のMicroPythonでMQTTを使ってみた](https://beta-notes.way-nifty.com/blog/2020/03/post-bdef2a.html)  

以上
