
2020/3/4

ESP32/ESP8266 MicroPython vs Web MQTT test
# ESP32/ESP8266 MicroPython vs Web MQTT test

## 概要
ESP32/ESP8266のMicroPythonとWeb-browserの間でMQTT通信をする。  
ESP32/ESP8266のMicroPython側は以下のリンクのスクリプトをそのまま使用するので、
ここではweb-browser側のhtmlのスクリプトについて記する。

[ESP32/ESP8266のMicroPythonでMQTTを使ってみた](https://beta-notes.way-nifty.com/blog/2020/03/post-bdef2a.html)  


## htmlスクリプト
web-browser側もスクリプトは以下の２つになる。   
それぞれMQTT01_test.py,MQTT02_test.pyと同機能のスクリプトになる。  
(browserは、chromeを推奨する)

MQTT_ESP01_webTest.html
```javascript


<html>

<script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>

<body>
<script>
//ESP#1
var topic_sub = 'notification';
var topic_pub = 'hello';

document.body.innerHTML = '';
var consout = 'MQTT over WebSockets Test(ESP01)'+'<br>'
document.body.innerHTML = consout;

//var mqtt = require('mqtt')
var client = mqtt.connect('ws://test.mosquitto.org:8081')

// subscribe Topic
client.subscribe(topic_sub);
//client.subscribe(topic_pub);  // for echo back test

client.on('message', function(topic, payload) {
  console.log([topic, payload].join(': '));
  consout += [topic, payload].join(': ')+'<br>'
  document.body.innerHTML = consout
  // disconnect
  //client.end();
});

// publish messages
var counter = 0;
var msg = '';
/**********************************
// infinite while loop does Not work! (can not go into event listener)
while (true) {
   msg = 'Hello #'+counter;
   console.log('publishing:: '+topic_pub+':'+msg);
   client.publish(topic_pub, msg);
   counter += 1
}
*********************************/
function repeatUnit() {
   msg = 'Hello #'+counter;
   console.log('publishing:: '+topic_pub+':'+msg);
   client.publish(topic_pub, msg);
   counter += 1
}
setInterval(repeatUnit, 100); // msec

</script>
</body>
</html>
```

MQTT_ESP02_webTest.html
```javascript

<html>

<script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>

<body>
<script>
//ESP#2
var topic_sub = 'hello';
var topic_pub = 'notification';

document.body.innerHTML = '';
var consout = 'MQTT over WebSockets Test(ESP02)'+'<br>'
document.body.innerHTML = consout;

//var mqtt = require('mqtt')
var client = mqtt.connect('ws://test.mosquitto.org:8081')

// subscribe Topic
client.subscribe(topic_sub);
//client.subscribe(topic_pub); // for echo back test

client.on('message', function(topic, payload) {
  console.log([topic, payload].join(': '));
  consout += [topic, payload].join(': ')+'<br>'
  document.body.innerHTML = consout
  // disconnect
  //client.end();
});

// publish messages
/****************************************
// infinite while loop does Not work! (can not go into event listener)
while (true) {
   client.publish(topic_pub, 'received');
}
****************************************/
function repeatUnit() {
   client.publish(topic_pub, 'received');
}
setInterval(repeatUnit, 100); // msec

</script>
</body>
</html>
```

単純な無限ループ(while (true))でpublishした場合、
イベントリスナーに制御が行かずにメッセージ受信ができなくなるので、
繰り返しは、setintervalを利用する必要がある。


## 実行画面

ESP32/SP8266のMicroPython側は
MQTT02_testを実行し、   
ブラウザー側は
MQTT_ESP01_webTest.htmlを実行すると  
以下のような実行画面になる：

MQTT_ESP01_webTest.htmlのブラウザー画面：
```

MQTT over WebSockets Test(ESP01)
notification: received
notification: received
notification: received
notification: received
notification: received
notification: received
notification: received
notification: received
notification: received
notification: received
notification: received
notification: received
```


ESP32/SP8266のMicroPython側は
MQTT01_testを実行し、  
ブラウザー側は
MQTT_ESP02_webTest.htmlを実行すると  
以下のような実行画面になる：

MQTT_ESP02_webTest.htmlのブラウザー画面：
```

MQTT over WebSockets Test(ESP02)
hello: Hello #1
hello: Hello #2
hello: Hello #3
hello: Hello #4
hello: Hello #5
hello: Hello #6
hello: Hello #7
hello: Hello #8
hello: Hello #9
hello: Hello #10
hello: Hello #11
hello: Hello #12
hello: Hello #13
hello: Hello #14
hello: Hello #15
```


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
[MQTT is a machine-to-machine (M2M)/"Internet of Things" ](http://mqtt.org/)  

[ESP32/ESP8266のMicroPythonでMQTTを使ってみた](https://beta-notes.way-nifty.com/blog/2020/03/post-bdef2a.html)  
[MQTT(over WebSocket)をブラウザーで使ってみた](https://beta-notes.way-nifty.com/blog/2020/03/post-ebcf2e.html)   


[MicroPython – Getting Started with MQTT on ESP32/ESP8266](https://randomnerdtutorials.com/micropython-mqtt-esp32-esp8266/)  

[ＥＳＰ３２－ＤｅｖＫｉｔＣ　ＥＳＰ－ＷＲＯＯＭ－３２開発ボード](http://akizukidenshi.com/catalog/g/gM-11819/)    
[MicroPython - Quick reference for the ESP32](https://docs.micropython.org/en/latest/esp32/quickref.html#)    
[Streaming Data from ESP32 using MicroPython and MQTT](https://github.com/gloveboxes/ESP32-MicroPython-BME280-MQTT-Sample/blob/master/Micropython.md)   
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)　　　

以上
