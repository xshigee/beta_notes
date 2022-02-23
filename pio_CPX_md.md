
2020/5/25

PlatformIO cli Circuit-Playground-Express
# PlatformIO cli Circuit-Playground-Express

## 概要
開発ツールPlatformIOを以下のCircuit-Playground-Expressで使う(arduino版)。    
VisualCodeのプラグインとしてPlatformIOを使用することができるが、ここでは、cliとしての使い方について記する。   
(ホストPCとしてはubuntuを想定している)

[Circuit Playground Express](https://www.switch-science.com/catalog/3666/)  

## PlatformIOのインストール
```bash

python3 -m venv pio_env
source pio_env/bin/activate

pip3 install platformio

インストール後も、本ツールを使用する場合
同じディレクトリで以下を実行する：  
source pio_env/bin/activate
# 「source」は、「.」でも良い

```
## 準備
以下を実行して、udevのrulesを登録する：
```bash


curl -fsSL https://raw.githubusercontent.com/platformio/platformio-core/master/scripts/99-platformio-udev.rules | sudo tee /etc/udev/rules.d/99-platformio-udev.rules

sudo udevadm control --reload-rules

sudo usermod -a -G dialout $USER
sudo usermod -a -G plugdev $USER

```

## テスト用プロジェクトを作成/実行する
```bash

#ターゲットボードのtarget名を検索する
# (ここでは circuitplayground を検索する)

$ pio boards | grep circuitplayground
#出力例：
adafruit_circuitplayground_m0  SAMD21G18A   48MHz        256KB    32KB   Adafruit Circuit Playground Express
 
#target名として「adafruit_circuitplayground_m0」が判明した

# プロジェクトdemoのディレクトリを作成する
mkdir demo	
cd demo
# 以下を実行して必要なファイルを作成する
pio init --board adafruit_circuitplayground_m0

nano platformio.ini
# platformio.iniを以下のように編集する：
```
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

[env:adafruit_circuitplayground_m0]
platform = atmelsam
board = adafruit_circuitplayground_m0
framework = arduino

upload_protocol = sam-ba
```

続き： 
```bash
#必要なライブラリーを用意する：
pio lib install "Adafruit Circuit Playground"

```
続き：

```bash
# テスト用のdemo.inoを作成する
nano src/demo.ino
以下の内容に編集する：
```
```c++

// Demo program for testing library and board - flip the switch to turn on/off buzzer

#include <Adafruit_CircuitPlayground.h>

// we light one pixel at a time, this is our counter
uint8_t pixeln = 0;

void setup() {
  Serial.begin(115200);
  Serial.println("Circuit Playground test!");

  CircuitPlayground.begin();
}

void loop() {
  // turn off speaker when not in use
  CircuitPlayground.speaker.enable(false);

  // test Red #13 LED
  CircuitPlayground.redLED(HIGH);
  delay(100);
  CircuitPlayground.redLED(LOW);

  /************* TEST CAPTOUCH */
  Serial.print("Capsense #3: "); Serial.println(CircuitPlayground.readCap(3));
  Serial.print("Capsense #2: "); Serial.println(CircuitPlayground.readCap(2));
  Serial.print("Capsense #0: "); Serial.println(CircuitPlayground.readCap(0));
  Serial.print("Capsense #1: "); Serial.println(CircuitPlayground.readCap(1));
  Serial.print("Capsense #12: "); Serial.println(CircuitPlayground.readCap(12));
  Serial.print("Capsense #6: "); Serial.println(CircuitPlayground.readCap(6));
  Serial.print("Capsense #9: "); Serial.println(CircuitPlayground.readCap(9));
  Serial.print("Capsense #10: "); Serial.println(CircuitPlayground.readCap(10));
  delay(10);

  /************* TEST SLIDE SWITCH */
  if (CircuitPlayground.slideSwitch()) {
    Serial.println("Slide to the left");
  } else {
    Serial.println("Slide to the right");
    CircuitPlayground.speaker.enable(true);
    CircuitPlayground.playTone(500 + pixeln * 500, 100);
  }
  delay(10);

  /************* TEST 10 NEOPIXELS */
  CircuitPlayground.setPixelColor(pixeln++, CircuitPlayground.colorWheel(25 * pixeln));
  if (pixeln == 11) {
    pixeln = 0;
    CircuitPlayground.clearPixels();
  }
  delay(10);

  /************* TEST BOTH BUTTONS */
  if (CircuitPlayground.leftButton()) {
    Serial.println("Left button pressed!");
  }
  if (CircuitPlayground.rightButton()) {
    Serial.println("Right button pressed!");
  }
  delay(10);

  /************* TEST LIGHT SENSOR */
  Serial.print("Light sensor: ");
  Serial.println(CircuitPlayground.lightSensor());
  delay(10);

  /************* TEST SOUND SENSOR */
  Serial.print("Sound sensor: ");
  Serial.println(CircuitPlayground.mic.soundPressureLevel(10));
  delay(10);

  /************* TEST ACCEL */
  // Display the results (acceleration is measured in m/s*s)
  Serial.print("X: "); Serial.print(CircuitPlayground.motionX());
  Serial.print(" \tY: "); Serial.print(CircuitPlayground.motionY());
  Serial.print(" \tZ: "); Serial.print(CircuitPlayground.motionZ());
  Serial.println(" m/s^2");
  delay(10);

  /************* TEST THERMISTOR */
  Serial.print("Temperature ");
  Serial.print(CircuitPlayground.temperature());
  Serial.println(" *C");

  Serial.print("===========================");
  delay(1000);

}
```
続き：
```bash

# build
pio run

# ボードをホストPCに接続する
# build&upload(flash)
pio run -t upload
# buildしないで書き込む場合は以下を実行する：
pio run -t nobuild -t upload -v
# -v は、詳細を表示するオプション

# 以上で、基本的な操作としては完了となる

```
書き込み後、ボード上のセンサー、スイッチ、LEDを使ったデモが起動する。   
センサーの値などはシリアル出力されるので「picocom /dev/ttyACM0 -b115200」を実行して表示する。


## 参考情報

[Circuit Playground Express](https://www.switch-science.com/catalog/3666/)   

[All in one library to control Adafruit's Circuit Playground](https://platformio.org/lib/show/602/Adafruit%20Circuit%20Playground)  


[Adafruit Circuit Playground Express - PINOUT](https://learn.adafruit.com/adafruit-circuit-playground-express/pinouts)  
[Adafruit Circuit Playground Express - Overview](https://learn.adafruit.com/adafruit-circuit-playground-express) 

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  


以上

