
2020/7/24:  
初版

board3 JSON lib test
# board3 JSON lib test

## 概要
ArduinoのJSONライブラリをテストする.    
(ホストPCとしてはubuntuを想定している)


## 外部ライブラリ(platformioの場合)
以下を実行してライブラリをインストールする：
```bash

# JSON lib インストール
#pio lib install "ArduinoJson"
pio lib install 64


```

## テスト・スケッチ1
  
src/JSON_deserialize_test.ino
```c++

// ArduinoJson - arduinojson.org
// Copyright Benoit Blanchon 2014-2020
// MIT License
//
// This example shows how to deserialize a JSON document with ArduinoJson.
//
// https://arduinojson.org/v6/example/parser/

#include <ArduinoJson.h>

void setup() {
  // Initialize serial port
  Serial.begin(115200);
  while (!Serial) continue;

  // Allocate the JSON document
  //
  // Inside the brackets, 200 is the capacity of the memory pool in bytes.
  // Don't forget to change this value to match your JSON document.
  // Use arduinojson.org/v6/assistant to compute the capacity.
  StaticJsonDocument<200> doc;

  // StaticJsonDocument<N> allocates memory on the stack, it can be
  // replaced by DynamicJsonDocument which allocates in the heap.
  //
  // DynamicJsonDocument doc(200);

  // JSON input string.
  //
  // Using a char[], as shown here, enables the "zero-copy" mode. This mode uses
  // the minimal amount of memory because the JsonDocument stores pointers to
  // the input buffer.
  // If you use another type of input, ArduinoJson must copy the strings from
  // the input to the JsonDocument, so you need to increase the capacity of the
  // JsonDocument.
  char json[] =
      "{\"sensor\":\"gps\",\"time\":1351824120,\"data\":[48.756080,2.302038]}";

  Serial.println();
  Serial.println("Source:");
  Serial.println(json);
  Serial.println();

  // Deserialize the JSON document
  DeserializationError error = deserializeJson(doc, json);

  // Test if parsing succeeds.
  if (error) {
    Serial.print(F("deserializeJson() failed: "));
    Serial.println(error.c_str());
    return;
  }

  // Fetch values.
  //
  // Most of the time, you can rely on the implicit casts.
  // In other case, you can do doc["time"].as<long>();
  const char* sensor = doc["sensor"];
  long time = doc["time"];
  double latitude = doc["data"][0];
  double longitude = doc["data"][1];

  // Print values.
  Serial.println("Deserialized(Parsed) data:");
  Serial.print("sensor:");
  Serial.println(sensor);
  Serial.print("time:");
  Serial.println(time);
  Serial.print("data[0]:");
  Serial.println(latitude, 6);
  Serial.print("data[1]:");
  Serial.println(longitude, 6);
}

void loop() {
  // not used in this example
}
```

書き込み後に「picocom /dev/ttyACM0 -b115200」または「picocom /dev/ttyUSB0 -b115200」で通信ソフトを起動すると以下のような出力が表示される：
```bash

$ picocom /dev/ttyACM0 -b115200

Terminal ready

Source:
{"sensor":"gps","time":1351824120,"data":[48.756080,2.302038]}

Deserialized(Parsed) data:
sensor:gps
time:1351824120
data[0]:48.756081
data[1]:2.302038
```

## テスト・スケッチ２
  
src/JSON_serialize_test.ino
```c++

// ArduinoJson - arduinojson.org
// Copyright Benoit Blanchon 2014-2020
// MIT License
//
// This example shows how to generate a JSON document with ArduinoJson.
//
// https://arduinojson.org/v6/example/generator/

#include <ArduinoJson.h>

void setup() {
  // Initialize Serial port
  Serial.begin(115200);
  while (!Serial) continue;

  // Allocate the JSON document
  //
  // Inside the brackets, 200 is the RAM allocated to this document.
  // Don't forget to change this value to match your requirement.
  // Use arduinojson.org/v6/assistant to compute the capacity.
  StaticJsonDocument<200> doc;

  // StaticJsonObject allocates memory on the stack, it can be
  // replaced by DynamicJsonDocument which allocates in the heap.
  //
  // DynamicJsonDocument  doc(200);

  // Add values in the document
  //
  doc["sensor"] = "gps";
  doc["time"] = 1351824120;

  // Add an array.
  //
  JsonArray data = doc.createNestedArray("data");
  data.add(48.756080);
  data.add(2.302038);

  // Generate the minified JSON and send it to the Serial port.
  //
  Serial.println();
  Serial.println("serialized output1:(serial directly)");
  serializeJson(doc, Serial);
  // The above line prints:
  // {"sensor":"gps","time":1351824120,"data":[48.756080,2.302038]}

  // Start a new line
  Serial.println();

  // Generate the prettified JSON and send it to the Serial port.
  //
  Serial.println();
  Serial.println("serialized output2:(serial directly)");
  serializeJsonPretty(doc, Serial);
  // The above line prints:
  // {
  //   "sensor": "gps",
  //   "time": 1351824120,
  //   "data": [
  //     48.756080,
  //     2.302038
  //   ]
  // }
  Serial.println();

  String serializedString;
  serializeJson(doc, serializedString);
  Serial.println();
  Serial.println("Serialized String:");
  Serial.println(serializedString);

  Serial.println();
  Serial.println("String Cat test");
  String strBuf1 = "Hello, World!";
  String strBuf2 = "CatTest";
  Serial.println(strBuf1.c_str());
  String strBuf3 = strBuf1+strBuf2;
  Serial.println(strBuf3);
}

void loop() {
  // not used in this example
}
```

書き込み後に「picocom /dev/ttyACM0 -b115200」または「picocom /dev/ttyUSB0 -b115200」で通信ソフトを起動すると以下のような出力が表示される：
```bash

$ picocom /dev/ttyACM0 -b115200

Terminal ready

serialized output1:(serial directly)
{"sensor":"gps","time":1351824120,"data":[48.75608,2.302038]}

serialized output2:(serial directly)
{
  "sensor": "gps",
  "time": 1351824120,
  "data": [
    48.75608,
    2.302038
  ]
}

Serialized String:
{"sensor":"gps","time":1351824120,"data":[48.75608,2.302038]}

String Cat test
Hello, World!
Hello, World!CatTest

```


## 参考情報

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

以上
