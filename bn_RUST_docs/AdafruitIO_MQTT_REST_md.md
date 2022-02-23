
2020/6/7

Adafruit IO MQTT REST-API
# Adafruit IO MQTT REST-API

## 概要
AdafruitのクラウドサービスであるAdafruit-IOのMQTT(REST-APIも含む)を利用してみる。
(ホストPCとしてはubuntuを想定している)


## 準備
adafruitのアカウントを用意する必要があるので
以下にアクセスしてアカウントを作る。(無償)  

https://accounts.adafruit.com  

アカウントを作成したら、Adafruit-IOにアクセスするための
アクセスキー(Adafruit IO KEY)を取得する必要があるので
以下にアクセスする：  

https://io.adafruit.com/USERNAME/dashboards  
(USERNAMEは作ったアカウントのユーザー名になる)  

ブラウザー画面の最上行の右端「Adafruit IO KEY」をクリックすると
アクセスキーが表示される。   

以下、このアクセスキー(IO_KEY)とアカウントのユーザー名(IO_USERNAME)を
プログラムに設定して使用する。

以下にアクセスライブラリ(Arduino,Python,CircuitPython,MicroPython,Ruby)が載っているが   

https://io.adafruit.com/api/docs/mqtt.html#client-libraries   

このうちPython3を使用してみる。

ライブラリのインストール：
```bash

python3 -m venv aio_env
source aio_env/bin/activate

pip3 install adafruit-io
```

## mqtt_time.py
MQTTを利用する形で時刻を提供しているので、それを利用するデモプログラム。

```bash

get https://raw.githubusercontent.com/adafruit/Adafruit_IO_Python/master/examples/mqtt/mqtt_time.py
# 取得したソースのIO_USERNAME,IO_KEYの部分を取得したものに置き換える。

```

出力例：
```

$ python3 mqtt_time.py 
* Subscribing to time/seconds
* Subscribing to time/millis
* Subscribing to time/ISO-8601
Connected to Adafruit IO!
	 Feed time received new value: 2020-06-05T03:19:29.402Z
	 Feed time received new value: 1591327169402
	 Feed time received new value: 1591327169
	 Feed time received new value: 2020-06-05T03:19:30.403Z
	 Feed time received new value: 1591327170403
	 Feed time received new value: 1591327170
	 Feed time received new value: 2020-06-05T03:19:31.404Z
	 Feed time received new value: 1591327171404
	 Feed time received new value: 1591327171
	 Feed time received new value: 2020-06-05T03:19:32.405Z
	 Feed time received new value: 1591327172405
	 Feed time received new value: 1591327172

```
## mqtt_client_class.py
トピックDemoFeedにsubscribe/publishするプログラム。

```bash 

wget https://raw.githubusercontent.com/adafruit/Adafruit_IO_Python/master/examples/mqtt/mqtt_client_class.py
# 取得したソースのIO_USERNAME,IO_KEYの部分を取得したものに置き換える。

```

出力例：
```

$ python3 mqtt_client_class.py 
Publishing a new message every 10 seconds (press Ctrl-C to quit)...
Connected to Adafruit IO!
Connected to Adafruit IO!  Listening for DemoFeed changes...
Publishing 6 to DemoFeed.　#送信
Feed DemoFeed received new value: 6　#受信
Publishing 60 to DemoFeed.　#送信
Feed DemoFeed received new value: 60　#受信
Publishing 63 to DemoFeed.　#送信
Feed DemoFeed received new value: 63　#受信
```

## mqtt_subscribe.py
MQTTの受信プログラム。

```bash

wget https://raw.githubusercontent.com/adafruit/Adafruit_IO_Python/master/examples/mqtt/mqtt_subscribe.py
# 取得したソースのIO_USERNAME,IO_KEYの部分を取得したものに置き換える。

```

出力例：
```

$ python3 mqtt_subscribe.py 
Connected to Adafruit IO!
Connected to Adafruit IO!  Listening for DemoFeed changes...
Subscribed to DemoFeed with QoS 0
Feed DemoFeed received new value: 5
Feed DemoFeed received new value: 97
Feed DemoFeed received new value: 18
Feed DemoFeed received new value: 44
Feed DemoFeed received new value: 87
Feed DemoFeed received new value: 60
Feed DemoFeed received new value: 1
```
別の端末から送信プログラムを動かしている必要がある。
(mqtt_client_class.pyでも良い)

## REST-APIによるアクセス(curl)
https://io.adafruit.com/api/docs/#get-feed のドキュメントより
REST-APIとしては以下であることが分かる：
```
GET /api/v2/{username}/feeds/{feed_key}
```
→  

curlを使用して実際に利用するには以下のようになる：
```
curl -X GET 'https://io.adafruit.com/api/v2/USERNAME/feeds/demofeed?X-AIO-Key=aio_xxxxx'
#USERNAMEでフィードキーdemofeedにアクセスする具体例になる。(aio_xxxxxは、aioのアクセスキーになる)

なお、''で囲まれた部分をURLとしてブラウザーに与えると同様の出力が表示される。

```
出力例：
```
{"username":"USERNAME","owner":{"id":44xxxx,"username":"USERNAME"},"id":138xxxx,"name":"DemoFeed","description":null,"license":null,
"history":true,"enabled":true,"visibility":"private","unit_type":null,"unit_symbol":null,
"last_value":"41","created_at":"2020-06-05T02:15:31Z","updated_at":"2020-06-06T15:16:53Z",
"status_notify":false,"status_timeout":4320,"status":"online","key":"demofeed","writable":true,
"group":{"id":36xxxx,"key":"default","name":"Default","user_id":44xxxx},"groups":[{"id":36xxxx,"key":"default","name":"Default","user_id":44xxxx}],"feed_webhook_receivers":[],"feed_status_changes":[]}
一部伏せ文字にしてある
```
上の情報から
フィードキーは、DemoFeedで
"updated_at":"2020-06-06T15:16:53Z"時点での
値が"last_value":"41"であることが分かる。

## REST-APIによるアクセス(html+javascript)

fetch_aio.html
```javascript

<!DOCTYPE html>
<head>
<meta charset="utf-8">
<title>Fetch Adafruit IO</title>
</head>
<body >
 <div id="text0">Fetch Adafruit IO</div>
 <div id="text1"></div>
<script>

let IO_USERNAME='username'
let IO_KEY='aio_xxxxxx'
let IO_URL='https://io.adafruit.com'
let FEED_KEY='demofeed'

let feed;
fetch(IO_URL+'/api/v2/'+IO_USERNAME+'/feeds/'+FEED_KEY+'?X-AIO-Key='+IO_KEY).then(function (response) {
    return response.text();
}).then(function (text) {
    console.log('RESULT: ' + text);
    feed=JSON.parse(text);
    console.log(feed['name']+': '+feed['last_value']+' @'+feed['updated_at']);
    text1.innerHTML = feed['name']+': '+feed['last_value']+' @'+feed['updated_at'];
});

</script>
</body>
</html>
```
以下の部分は、自分の環境にあわせて書き換える：
```javascript

let IO_USERNAME='username'
let IO_KEY='aio_xxxxxx'
```

アクセス時のweb画面(例)
```

Fetch Adafruit IO
DemoFeed: 94 @2020-06-07T02:20:30Z
```
別端末で送信プログラム(mqtt_client_class.py)を動かしておく。
ブラウザー画面をリフレッシュするたびに、最新の(受信した)値が表示される。
ブラウザーのconsoleにもアクセスした情報が出力される。

## 参考情報

[Client Libraries](https://io.adafruit.com/api/docs/#client-libraries)  
[Adafruit IO HTTP API](https://io.adafruit.com/api/docs/#adafruit-io-http-api)  


[JavaScriptのFetch APIを利用してリクエストを送信する](https://sbfl.net/blog/2017/01/29/fetch-api/)  
[javascript.info - Fetch](https://ja.javascript.info/fetch)  
[fetch の使い方](https://javascript.keicode.com/newjs/fetch.php)  

以上

