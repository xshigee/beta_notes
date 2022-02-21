
2020/5/16

PlatformIO Nucleo-F303K8 AQM0802
# PlatformIO Nucleo-F303K8 AQM0802

## 概要
Nucleo-F303K8でADCの値を以下のLCD(AQM0802)(i2c)に表示する。(Arduino版)  
(ホストPCとしてはubuntuを想定している)

[Ｉ２Ｃ接続小型ＬＣＤモジュール（８×２行）ピッチ変換キット(AE-AQM0802)](http://akizukidenshi.com/catalog/g/gK-06795/)  
(基板上のプルアップは有効にしている）

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

## 接続
以下の表のように、ボードとLCDを接続する：

| F303K8 | AE-AQM0802 |
| :---: | :---: |	
| 3V3 | VDD |
| RESET | NC |
| D5 | SCL |
| D4 | SDA |
| GND | GND |

NC: None Connection


## テスト用プロジェクト sample-adc1 を作成/実行する
```bash

#ターゲットボードのtarget名を検索する
# (ここではNucleo-F303K8を検索する)

$ pio boards | grep -i F303K8
#出力例：
nucleo_f303k8              STM32F303K8T6   72MHz        64KB      12KB      ST Nucleo F303K8

nucleo_f303k8


#target名として「nucleo_f303k8」が判明した

# プロジェクトnucleo-adc1 のディレクトリを作成する
mkdir nucleo-adc1	
cd sample-nucleo
# 以下を実行して必要なファイルを作成する
pio init --board nucleo_f303k8

# platformioを編集する
nano platfomio.ini
#以下の内容に編集する：
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

[env:nucleo_f303k8]
platform = ststm32
board = nucleo_f303k8
framework = arduino
upload_protocol = stlink
debug_took = stlink
```
続き：
```bash
# LCD(ST7032)ライブラリをインストールする
pio lib install 445

# テスト用のmain.cppを作成する
nano src/main.cpp
以下の内容に編集する：
```
```c++

#include <Arduino.h>

#include <Wire.h>
#include <ST7032.h>

// initialize the library with the numbers of the interface pins
//LiquidCrystal lcd(12, 11, 5, 4, 3, 2);
ST7032 lcd;

int a0;
int a1;
int a2;
int a3;

void setup() {
  Serial.begin(115200); //  setup serial
  // set up the LCD's number of columns and rows: 
  lcd.begin(8, 2);
  lcd.setContrast(30); // should be changed to fit your board
  // Print a message to the LCD.
  lcd.print("HelloWLD");
  Serial.println("Hello World.");
}

void loop() {
  a0 = analogRead(0);
  a1 = analogRead(1);
  a2 = analogRead(2);
  a3 = analogRead(3);

  // set the cursor to column 0, line 1
  // (note: line 1 is the second row, since counting begins with 0):
  lcd.setCursor(0, 0);
  lcd.print("A0: "+String(a0)+"     "); // +"     " to clear garbage
  float v1 = a1 * 3.3 / 1023.0;
  lcd.setCursor(0, 1);
  lcd.print("V1: "+String(v1)+"     "); // show voltage
//
  Serial.println("A0: "+String(a0));
  Serial.println("A1: "+String(a1));
  Serial.println("A2: "+String(a2));
  Serial.println("A3: "+String(a3));
  Serial.println("------------------");
  delay(500); 

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

# 注意：エラーが出て書き込めない場合
# いったん、USBの接続を切って再接続すると良いようだ。

```
書き込み後、LCDの１行目にA0の値が２行目にA1の値を電圧に変換した値が表示される。
シリアル出力(/dev/ttyACM0)には、A0,A1,A2,A3の値が出力される。

## 参考情報

[Library for ST7032i display](https://platformio.org/lib/show/445/ST7032)  

[NUCLEO-F303K8](https://os.mbed.com/platforms/ST-Nucleo-F303K8/)  
[開発ツールPlatformIOをcli(comand line interface)で使う(Nucleo版)](https://beta-notes.way-nifty.com/blog/2020/04/post-b394b4.html)

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

以上

