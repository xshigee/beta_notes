
2020/8/5+   

PlatformIO cli M5ATOM
# PlatformIO cli M5ATOM

## 概要
以下のM5Atomを開発ツールPlatformIOで使う(M5Atom/Arduino版)。
(ホストPCとしてはubuntuを想定している)

[ATOM Lite](https://docs.m5stack.com/#/en/core/atom_lite)  
[ATOM Matrix](https://docs.m5stack.com/#/en/core/atom_matrix)  

## Peripherals Pin Map
Lite:
| Func | GPIO |
| ---: | :--- |
| RGB Led(Neo) | G27 |
| Btn |	G39 |
| IR | G12 |

Matrix:
| Func | GPIO |
| ---: | :--- |
| Neo | G27 |
| Btn |	G39 |
| IR | G12 |
| CLK(MPU6886) | G21 |
| SDA(MPU6886) | GP25 |

Grove Interface
| GND | 5V | G26 | G32 |
| :---: | :---: | :---: | :---: |
| GND | 5V | SDA | SCL |


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
過去に設定してあったとしても、rulesが更新されている場合があるので、再設定したほうが良い。

## テスト用プロジェクト sample を作成/実行する
```
# プロジェクト sample のディレクトリを作成する
mkdir sample	
cd sample
# 以下を実行して必要なファイルを作成する
pio init --board m5stick-c

# platformをupdateする
pio platform update

nano platformio.ini
以下にように編集する：
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

[env:esp32dev]
platform = espressif32
board = m5stick-c
framework = arduino

monitor_speed = 115200

lib_deps =
    # use M5Atom lib
    3113
    # use "FastLED"
    126

lib_ldf_mode = deep+
```
```bash
# テスト用のbutton.inoを作成する
nano src/button.ino
以下のように編集する：
```
```c++

/****************************************************************
 * 
 * This Example is used to test button
 * 
 * Arduino tools Setting 
 * -board : M5StickC
 * -Upload Speed: 115200 / 750000 / 1500000
 * 
****************************************************************/

#include "M5Atom.h"

uint8_t DisBuff[2 + 5 * 5 * 3];

void setBuff(uint8_t Rdata, uint8_t Gdata, uint8_t Bdata)
{
    DisBuff[0] = 0x05;
    DisBuff[1] = 0x05;
    for (int i = 0; i < 25; i++)
    {
        DisBuff[2 + i * 3 + 0] = Rdata;
        DisBuff[2 + i * 3 + 1] = Gdata;
        DisBuff[2 + i * 3 + 2] = Bdata;
    }
}

void setup()
{
    M5.begin(true, false, true);
    delay(10);
    setBuff(0xff, 0x00, 0x00);
    M5.dis.displaybuff(DisBuff);
}

uint8_t FSM = 0;

void loop()
{
    if (M5.Btn.wasPressed())
    {

        switch (FSM)
        {
        case 0:
            setBuff(0x40, 0x00, 0x00);
            break;
        case 1:
            setBuff(0x00, 0x40, 0x00);
            break;
        case 2:
            setBuff(0x00, 0x00, 0x40);
            break;
        case 3:
            setBuff(0x20, 0x20, 0x20);
            break;
        default:
            break;
        }
        M5.dis.displaybuff(DisBuff);

        FSM++;
        if (FSM >= 4)
        {
            FSM = 0;
        }
    }

    delay(50);
    M5.update();
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
書き込み後、ボタンを押す度に、LEDの色が変化する。
(本スケッチは、Matrix/Lite兼用になっていて、Matrixの場合、5x5のLEDが同じ色で光る。Liteの場合、一つのLEDが光る)

これ以降、別のプログラムを動かすときは
sampleのディレクトリをまるごと
コピーして別のプロジェクトのディレクトリを作り
そこにプログラム(.ino)を置く。  

例：  

```bash

cp sample m5a_proj01
cd m5a_proj01
...
```

## MPU6886を利用するスケッチ#1(Matrix限定)
src/mpu6886_2.ino
```c++

#include "M5Atom.h"
#define SIG_MAX (4096)

uint8_t DisBuff[2 + 5 * 5 * 3];
int16_t adX,adY,adZ;

void setBuffP(uint8_t posData, uint8_t Rdata, uint8_t Gdata, uint8_t Bdata)
{
    DisBuff[2 + posData * 3 + 0] = Rdata;
    DisBuff[2 + posData * 3 + 1] = Gdata;
    DisBuff[2 + posData * 3 + 2] = Bdata;
}
void setBuff(uint8_t Rdata, uint8_t Gdata, uint8_t Bdata)
{
    for (uint8_t i = 0; i < 25; i++)
        setBuffP(i, Rdata, Gdata, Bdata);
}
void shftBuff()
{
    for (uint8_t i = 24; i > 0; i--)
    {
        for (uint8_t j = 0; j < 3; j++)
            DisBuff[2 + i * 3 + j] = DisBuff[2 + (i-1) * 3 + j];
    }
}

void setup()
{
    DisBuff[0] = 0x05;
    DisBuff[1] = 0x05;
    M5.begin(false, true, true);
    delay(10);
    M5.IMU.Init();
    setBuff(0x20, 0x20, 0x20);
    M5.dis.displaybuff(DisBuff);
    //
    Serial.begin(115200);
}

void loop()
{
    M5.update();
    if (M5.Btn.read()==0)
    {
        M5.IMU.getAccelAdc(&adX, &adY, &adZ);
        int r = min(max((int)map(adX,-SIG_MAX,SIG_MAX,0,255),0),255);
        int g = min(max((int)map(adY,-SIG_MAX,SIG_MAX,0,255),0),255);
        int b = min(max((int)map(adZ,-SIG_MAX,SIG_MAX,0,255),0),255);
        shftBuff();
        setBuffP(0,r,g,b);
        M5.dis.displaybuff(DisBuff);
        //
        Serial.printf("x,y,z: %d,%d,%d\r\n", adX, adY, adZ);

    }
    delay(20);
}
```
書き込み実行し、M5Atomを傾けるとそれに応じて5x5のLEDの色が変化する。

## MPU6886を利用するスケッチ#2(Matrix限定)
src/mpu6886_3.ino
```c++

#include "M5Atom.h"

float accX = 0, accY = 0, accZ = 0;
float gyroX = 0, gyroY = 0, gyroZ = 0;
float temp = 0;
bool IMU6886Flag = false;

void setup()
{
    M5.begin(true, false, true);

    if (M5.IMU.Init() != 0)
        IMU6886Flag = false;
    else
        IMU6886Flag = true;
}

void loop()
{

    if (IMU6886Flag == true)
    {
        M5.IMU.getGyroData(&gyroX, &gyroY, &gyroZ);
        M5.IMU.getAccelData(&accX, &accY, &accZ);
        M5.IMU.getTempData(&temp);

        Serial.printf("%.2f,%.2f,%.2f o/s \r\n", gyroX, gyroY, gyroZ);
        Serial.printf("%.2f,%.2f,%.2f mg\r\n", accX * 1000, accY * 1000, accZ * 1000);
        Serial.printf("Temperature : %.2f C \r\n", temp);
    }
    delay(500);
    M5.update();
}
```


## WiFi対応スケッチ
ESP32のWiFi対応スケッチと互換性があるので、以下をソースの先頭に入れると動作するようだ。
```

#ifdef M5ATOM
#include "M5Atom.h"
#define ESP32
#endif
```

したがって、以下のスケッチの先頭に上のコードを入れるとM5Atomでも動作するようになる：  
・[Wio-Terminal/ESP8622/ESP32ボードを共通のスケッチでWorld Time APIを使う(WorldTimeAPI編)](https://beta-notes.way-nifty.com/blog/2020/08/post-713652.html)  
・[Wio-Terminal/ESP8622/ESP32ボードを共通のスケッチで動かす(HTTP-ACCESS編)](https://beta-notes.way-nifty.com/blog/2020/08/post-2f0b6e.html)  
・[Wio-Terminal/ESP8622/ESP32ボードを共通のスケッチで動かす(NTP-CLIENT編)](https://beta-notes.way-nifty.com/blog/2020/08/post-3484c0.html)  
・[Wio-Terminal/ESP8622/ESP32ボードを共通のスケッチで動かす(REST-API2編)](https://beta-notes.way-nifty.com/blog/2020/07/post-7d77cd.html)  
・[Wio-Terminal/ESP8622/ESP32ボードを共通のスケッチで動かす(OSC編)](https://beta-notes.way-nifty.com/blog/2020/07/post-8d74a1.html)    
・[Wio-Terminal/ESP8622/ESP32ボードを共通のスケッチで動かす(MQTT編)](https://beta-notes.way-nifty.com/blog/2020/07/post-f0f859.html)  


## 参考情報

Atom pixel tool  
wget https://m5stack.oss-cn-shenzhen.aliyuncs.com/resource/software/AtomPixTool.exe  
windowsのプログラムだがlinuxのwineでも動作するようだ。  
マトリックスのデザインを作成でき、それを保存すると、そのデータのC言語ソースができあがる   

Display API Document:  
https://github.com/m5stack/M5Atom  

サンプル・スケッチ：  
git clone https://github.com/m5stack/M5Atom.git  
git clone https://github.com/hajimef/m5atom-matrix-samples.git  
[M5AtomをPlatformIOで動かす-ライブラリインストールから加速度取得まで](https://qiita.com/ELIXIR/items/663114ddb2d5ab621c66)   

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  


以上

