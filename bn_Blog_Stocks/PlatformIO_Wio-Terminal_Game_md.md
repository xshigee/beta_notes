
2020/7/29   

PlatformIO cli Wio Terminal Space Shooter
# PlatformIO cli Wio Terminal Space Shooter

## 概要
以下にあるWio-Terminalの「Space Shooter」(game)をplatformioでコンパイルして実行する(Seeeduino-Wio-Terminal/Arduino版)  
platformioのインストールについては
「[開発ツールPlatformIOをcliで使う(Seeeduino-Wio-Terminal/Arduino版)](https://beta-notes.way-nifty.com/blog/2020/07/post-3f0abb.html)」を参照のこと。(ホストPCとしてはubuntuを想定している)

[Wio Space Shooter](https://macsbug.wordpress.com/2020/05/27/wio-space-shooter/)  

## プロジェクト wiot-game のディレクトリを作成する

```
mkdir wiot-game	
cd wiot-game
# 以下を実行して必要なファイルを作成する
pio init --board seeed_wio_terminal

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

[env:seeed_wio_terminal]
platform = atmelsam
board = seeed_wio_terminal
framework = arduino

upload_protocol = sam-ba

lib_ldf_mode = deep+
```

## 該当ライブラリとスケッチのダウンロード
```bash

cd wiot-game
cd src

wget https://macsbug.files.wordpress.com/2020/05/wio_space_shooter.zip_.pdf
mv wio_space_shooter.zip_.pdf wio_space_shooter.zip
unzip wio_space_shooter.zip
cp Wio_Space_Shooter/*.ino src/
cp Wio_Space_Shooter/*.h src/

#または、別のゲームをコンパイルする場合、以下を実行する
wget https://macsbug.files.wordpress.com/2020/05/wio_clappy_shooter.zip_.pdf
mv wio_clappy_shooter.zip_.pdf wio_clappy_shooter.zip
unzip wio_clappy_shooter.zip
cp Wio_Clappy_Shooter/*.ino src/
cp Wio_Clappy_Shooter/*.h src/
```

## 追加するdefine
Arduino-IDEでは未定義にならないが、platformioでは
未定義(原因不明)になるので、以下を*.inoのソースの先頭に追加する。
```

// #define for wio terminal (for platformio)
#define WIO_KEY_A 28
#define WIO_KEY_B 29
#define WIO_KEY_C 30
#define WIO_5S_UP 31
#define WIO_5S_DOWN 34
#define WIO_5S_LEFT 32
#define WIO_5S_RIGHT 33
#define WIO_5S_PRESS 35
#define WIO_BUZZER 12
//------------------

```

## Patch
*.inoのソースにミスがあるので以下の修正を行なう：  
(ミススペル)
```

  pinMode(WIO_5S_DOWM,  INPUT_PULLUP); // 
→
  pinMode(WIO_5S_DOWN,  INPUT_PULLUP); // 
```

## コンパイル/書き込み/実行
```

# ボードをホストPCに接続する
# build&upload(flash)
pio run -t upload
# buildしないで書き込む場合は以下を実行する：
pio run -t nobuild -t upload -v
# -v は、詳細を表示するオプション
```
書き込み後、スペースインベーダー風のゲームが起動する。


## 実行するスケッチ(参考)
src/Wio_Space_Shooter.ino
```c++


//======================== intro =======================================
//      Space Shooter, basically knock-off Space Invaders
//             and also maybe a bit of Galaga
//   Written by Tyler Edwards for the Hackerbox #0020 badge
//  But should work on any ESP32 and Adafruit ILI9341 display
//        I am sorry for the messy code, you'll just
//                  have to live with it
//      Tyler on GitHub: https://github.com/HailTheBDFL/
//          Hackerboxes: http://hackerboxes.com/
//=========================== setup ===================================
// Wio Space Shooter : 2020.05.27 : Transplant by macsbug
//  https://macsbug.wordpress.com/2020/05/27/wio-space-shooter/
// Controller   : KEY_B, 5S_PRESS = START, SHOOT.
// Controller   : KEY_C, 5S_LEFT = LEFT.  KEY_A, 5S_RIGHT =RIGHT.  
// Space Shooter with M5STACK : 2018.01.12 Transplant by macsbug
// Controller   : Buttons A = LEFT, B = RIGHT, C = START or SHOOTING
// Github:https://macsbug.wordpress.com/2018/01/12/esp32-spaceshooter-with-m5stack/
//===================================================================
// #define for wio terminal (for platformio)
#define WIO_KEY_A 28
#define WIO_KEY_B 29
#define WIO_KEY_C 30
#define WIO_5S_UP 31
#define WIO_5S_DOWN 34
#define WIO_5S_LEFT 32
#define WIO_5S_RIGHT 33
#define WIO_5S_PRESS 35
#define WIO_BUZZER 12
//------------------
#include "Free_Fonts.h"
#include <TFT_eSPI.h>
TFT_eSPI tft = TFT_eSPI();
//============================= game variables =========================
unsigned long offsetM  = 0;
unsigned long offsetT  = 0;
unsigned long offsetF  = 0;
unsigned long offsetB  = 0;
unsigned long offsetA  = 0;
unsigned long offsetAF = 0;
unsigned long offsetAB = 0;
unsigned long offsetS  = 0;
int threshold = 40;
boolean startPrinted = false;
boolean beginGame = false;
boolean beginGame2 = true;
boolean play = false;
int score = 0;
int scoreInc = 10;
int level = 1;
//---------------------Player---------------------------------------
int shipX = 147;
int shipY = 190;
int oldShipX = 0;
int oldShipY = 0;
int changeShipX = 0;
int changeShipY = 0;
int shipSpeed = 50;
boolean doSplode = false;
boolean fire = false;
int fFireX[5] = {0, 0, 0, 0, 0};
int fFireY[5] = {0, 0, 0, 0, 0};
int fFireAge[5] = {0, 0, 0, 0, 0};
//--------------------------Aliens----------------------------------
boolean alienLive[18];
int alienLiveCount = 18;
int alienX = 7;
int alienY = 25;
int oldAlienX = 7;
int oldAlienY = 25;
int changeAlienX = 6;
int changeAlienY = 0;
int alienSpeed = 200;
int oldAlienSpeed;
int aFireX[5];
int aFireY[5];
boolean aFireAge[5];
int chanceOfFire = 2;
//================================ bitmaps ========================
//your starship
const int shipImgW = 14; const int shipImgH = 16;
char shipImg[] = 
//xxxxxxxxxxxxxx--------------xxxxxxxxxxxxxx--------------
 "ZZZZZZWWZZZZZZZZZZZYWWYZZZZZZZZZZWWWWZZZZZZZZZZWWWWZZZZZ"
 "ZZZZWWWWWWZZZZZZZZWWWWWWZZZZZYZZWWWWWWZZYZZYZZWWWWWWZZYZ"
 "WWZZWWBBWWZZWWWWZZWBBBBWZZWWWWZWWBBBBWWZWWWWZWWWWWWWWZWW"
 "WWWWWWWWWWWWWWWRWWWWWWWWWWRWZZWWWWWWWWWWZZZZZWRRWWRRWZZZ";
//flames
const int flamesImgW = 12; const int flamesImgH = 6;
char flamesImg[] = 
//xxxxxxxxxxxxxx--------------xxxxxxxxxxxxxx--------------
 "RZZZZZZZZZZRRZRYYRRYYRZRRZRYYRRYYRZRZZRYRZZRYRZZZZZRZZZZ"
 "RZZZZZZRZZZZRZZZ";
//alien
const int alienImgW = 14; const int alienImgH = 11;
char alienImg[] = 
//xxxxxxxxxxxxxx--------------xxxxxxxxxxxxxx--------------
 "GGGZZZZZZZZGGGZZZGZZZZZZGZZZZZGGGGGGGGGGZZZGGGGGGGGGGGGZ"
 "GGGZGGGGGGZGGGGGGZZGGGGZZGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG"
 "GZZZGGZZGGZZZGZGGZGGZZGGZGGZZZZZZGZZGZZZZZ";
//ship 'sploded
const int splodedImgW = 14; const int splodedImgH = 16;
char splodedImg[] = 
//xxxxxxxxxxxxxx--------------xxxxxxxxxxxxxx--------------
 "ZZZZZZWWZZZZZZZZZZRYWWYRZZZYZZZRRWWRRRRZRWYZRRRRRYYRRRZW"
 "YZRYRYYRYYRRRZWWRYYYRYYYYYRZWWRYYRYRYYYYRRWWRYYYRWRYBRRZ"
 "RRRYRRWWWRYRWZZRYYRRBBWRYRWWZZRYYBBBRRYBWWRZZRYYYRRYYZZW"
 "ZRRWRYYRBYRZZWZZRYBRYYYYYRRZZRWWYYYWWRRRZZZZWRRWWRRRWZZZ";
//=============================== setup and loop ==================
void setup() {
  memset(alienLive, true, 18);
  memset(aFireX,   0, 5);
  memset(aFireY,   0, 5);
  memset(aFireAge, 0, 5);
  tft.begin();
  tft.setRotation(3);
  tft.fillScreen(TFT_BLACK);
  tft.setFreeFont(FM24);
  tft.setTextColor(TFT_GREEN);
  randomSeed(analogRead(6));
  //-----------------------------------// BUTTON & 5 Way Swicth ---
  pinMode(WIO_KEY_A,    INPUT_PULLUP); // RIGHT
  pinMode(WIO_KEY_B,    INPUT_PULLUP); // CENTER
  pinMode(WIO_KEY_C,    INPUT_PULLUP); // LEFT
  pinMode(WIO_5S_UP,    INPUT_PULLUP); // 
  //pinMode(WIO_5S_DOWM,  INPUT_PULLUP); // 
  pinMode(WIO_5S_DOWN,  INPUT_PULLUP); // 
  pinMode(WIO_5S_LEFT,  INPUT_PULLUP); // 
  pinMode(WIO_5S_RIGHT, INPUT_PULLUP); // 
  pinMode(WIO_5S_PRESS, INPUT_PULLUP); // 
  //-----------------------------------// 
}
//==================================================================
void loop() {
  if(digitalRead(WIO_KEY_C)== LOW || 
     digitalRead(WIO_5S_LEFT )== LOW){left  ();}
  if(digitalRead(WIO_KEY_A)== LOW || 
     digitalRead(WIO_5S_RIGHT)== LOW){right ();}
  if(digitalRead(WIO_KEY_B)== LOW || 
     digitalRead(WIO_5S_PRESS)== LOW){select();}
  //-------------Start Screen--------------
  if (millis() - offsetS >= 900 and !beginGame) {
    if (!startPrinted) {
      tft.setTextColor(TFT_GREEN);
      tft.setCursor(87, 105);tft.setFreeFont(FM18);
      tft.print(">START<");
      startPrinted = true;
      offsetS = millis();
    } else {
      tft.fillRect(89, 85, 143, 22, TFT_BLACK);
      startPrinted = false;
      offsetS = millis();
    }
  }
  if (beginGame and beginGame2) {
    tft.fillRect(89, 85, 143, 22, TFT_BLACK);
    beginGame2 = false;
    play = true;
  }
  //-------------Player-----------------------------------------------
  if (millis() - offsetM >= shipSpeed and play) {
    moveShip();
    offsetM = millis();
  }
  if (oldShipX != shipX or oldShipY != shipY) {
    tft.fillRect(oldShipX, oldShipY, 28, 44, TFT_BLACK);
    oldShipX = shipX;
    oldShipY = shipY;
    drawBitmap(shipImg, shipImgW, shipImgH, shipX, shipY, 2);
  }
  if (fire and play) { fireDaLazer();}
  if (millis() - offsetB >= 50) {
   for (int i = 0; i < 5; i++) {
    if (fFireAge[i] < 20 and fFireAge[i] > 0){keepFirinDaLazer(i);}
    if (fFireAge[i] == 20) { stopFirinDaLazer(i);}
   }
   offsetB = millis();
  }  
  if (millis() - offsetT > 50){changeShipX = 0;changeShipY = 0;}
  //---------------Aliens--------------------------------------------
  if (millis() - offsetA >= alienSpeed and play) {
    moveAliens(); offsetA = millis();
  }
  if (findAlienX(5) >= 294){changeAlienX = -3;changeAlienY = 7;}
  if (alienX <= 6){changeAlienX = 3; changeAlienY = 7;}
  alienLiveCount = 0;
  for (int i = 0; i < 18; i++) {
   if (alienLive[i]) {
    alienLiveCount += 1;
     if (alienShot(i)) {
      tft.fillRect(findOldAlienX(i),findOldAlienY(i),28,22,TFT_BLACK);
      alienLiveCount -= 1;
      alienLive[i] = false;
      score += scoreInc;
     }
     if (onPlayer(i) or exceedBoundary(i)) {
      gameOver();
    }
   }
  }
  if (alienLiveCount == 1) {
    oldAlienSpeed = alienSpeed;
    if (alienSpeed > 50) {
      alienSpeed -= 10;
    }
    else {
      alienSpeed = 20;
    }
  }
  if (alienLiveCount == 0) {
    levelUp();
  }
}
// functions =======================================================
void gameOver() {
  play = false;
  if (doSplode) {
    drawBitmap(splodedImg,splodedImgW,splodedImgH,shipX,shipY,2);
  }
  tft.fillScreen(TFT_BLACK);
  drawScore(false);
  delay(1000);
  tft.setCursor(17,168);tft.setFreeFont(FM9);
  tft.print("(Reset device to replay)");
  while (1) { }
}
//==================================================================
void drawScore(boolean win) {
  tft.setCursor(53, 40);tft.setFreeFont(FM18);
  tft.setTextColor(TFT_WHITE);
  if (win) {
         tft.print("LEVEL UP!");
  }
  else { tft.print("GAME OVER");
  }
  for (;millis() - offsetM <= 1000;)
  tft.setCursor(59, 89);tft.setFreeFont(FM18);
  tft.print("Score: "); tft.print(score);
  offsetM = millis();
  for (;millis() - offsetM <= 1000;) {
  }
  tft.setCursor(59, 128);tft.setFreeFont(FM18);
  tft.print("Level: "); tft.print(level);
}
//==================================================================
void levelUp() {
  play = false;
  memset(alienLive, true, 18);
  memset(aFireX, 0, 5);
  memset(aFireY, 0, 5);
  memset(aFireAge, 0, 5);
  alienX = 7;
  alienY = 25;
  oldAlienX = 7;
  oldAlienY = 25;
  alienSpeed = oldAlienSpeed;
  if (alienSpeed > 100) {
    alienSpeed -= 10; chanceOfFire -= 10;
  }
  else if (alienSpeed > 50) {
    alienSpeed -= 10; chanceOfFire -=5;
  }
  else if (alienSpeed > 25) {
    alienSpeed -= 5; chanceOfFire -=1;
  }
  score += 50;     scoreInc += 5;
  changeShipX = 0; changeShipY = 0; 
  for (unsigned long i = millis(); millis() - i <= 1600;) {
   if (millis() - offsetM >= 20) {
    tft.fillRect(oldShipX, oldShipY, 28, 44, TFT_BLACK);
     drawBitmap(shipImg,shipImgW,shipImgH,shipX,shipY,2);
     drawBitmap(flamesImg,flamesImgW,flamesImgH,shipX+1,shipY+32,2);
     oldShipX = shipX; oldShipY = shipY;
     shipY -= 6;
     offsetM = millis();
   }
  }
  drawScore(true);
  level += 1;
  shipX = 147;
  shipY = 190;
  for (; millis() - offsetM <= 4000;) {
  }
  tft.fillScreen(TFT_BLACK);
  offsetM = millis();
  play = true;
}
//==================================================================
boolean alienShot(int num) {
  for (int i=0; i < 5; i++) {
    if (fFireAge[i] < 20 and fFireAge[i] > 0) {
      if (fFireX[i] > findAlienX(num) - 4 and fFireX[i] < 
         findAlienX(num) + 28 and fFireY[i] < findAlienY(num) + 
         22 and fFireY[i] > findAlienY(num) + 4) {
        fFireAge[i] = 20;
        return true;
      }
    }
  }
  return false;
}
//==================================================================
boolean onPlayer(int num) {
  if (findAlienX(num) - shipX < 24 and findAlienX(num) - 
      shipX > -28 and findAlienY(num) - shipY < 32 and 
      findAlienY(num) - shipY > -22) {
    doSplode = true;
    return true;
  } else { return false;}
}
//==================================================================
boolean exceedBoundary(int num) {
  if (findAlienY(num) > 218) { return true;
  } else { return false;
  }
}
//==================================================================
void moveAliens() {
  for (int i = 0; i < 18; i++) {
   if (alienLive[i]) {
    tft.fillRect(findOldAlienX(i),findOldAlienY(i),28,22,TFT_BLACK);
    drawBitmap(alienImg,alienImgW,alienImgH,findAlienX(i),
               findAlienY(i),2);           
   }
  }
  oldAlienX = alienX; oldAlienY = alienY;
  alienX += changeAlienX; alienY += changeAlienY;
  if (changeAlienY != 0) { changeAlienY = 0; }
}
//==================================================================
int findAlienX   (int num){ return alienX    + 42*(num % 6);}
//==================================================================
int findAlienY   (int num){ return alienY    + 33*(num / 6);}
//==================================================================
int findOldAlienX(int num){ return oldAlienX + 42*(num % 6);}
//==================================================================
int findOldAlienY(int num){ return oldAlienY + 33*(num / 6);}
//---------------------------Player---------------------------------
void fireDaLazer() {
  int bulletNo = -1;
  for (int i = 0; i < 4; i++) {
    if (fFireAge[i] == 0) { bulletNo = i;}
  }
  if (bulletNo != -1) {
   fFireAge[bulletNo] = 1;
   fFireX[bulletNo]   = shipX + 13;
   fFireY[bulletNo]   = shipY -  4;
   tft.fillRect(fFireX[bulletNo],fFireY[bulletNo],4,3,TFT_MAGENTA);
  }
  fire = false;
}
//==================================================================
void keepFirinDaLazer(int bulletNo) {
  tft.fillRect(fFireX[bulletNo],fFireY[bulletNo],4,4,TFT_BLACK);
  fFireY[bulletNo] -= 8;
  tft.fillRect(fFireX[bulletNo],fFireY[bulletNo],4,4,TFT_MAGENTA);
  fFireAge[bulletNo] += 1;
}
//==================================================================
void stopFirinDaLazer(int bulletNo) {
  tft.fillRect(fFireX[bulletNo],fFireY[bulletNo],4,4,TFT_BLACK);
  fFireAge[bulletNo] = 0;
}
//==================================================================
void moveShip() {
  if (shipX + changeShipX < 288 and shipX + changeShipX > 
      6 and changeShipX != 0){
    shipX += changeShipX;
  }
  if (shipY + changeShipY > 24 and shipY + changeShipY < 
      192 and changeShipY != 0){
    shipY += changeShipY;
  }
  if (oldShipX != shipX or oldShipY != shipY) {
    tft.fillRect(oldShipX, oldShipY, 28, 44, TFT_BLACK);
    oldShipX = shipX; oldShipY = shipY;
    drawBitmap(shipImg, shipImgW, shipImgH, shipX, shipY, 2);
  }
}
//==================================================================
void drawBitmap(char img[],int imgW,int imgH,int x,int y,int scale){
  uint16_t cellColor;
  char curPix;
  for (int i = 0; i < imgW*imgH; i++) {
    curPix = img[i];
    if (curPix == 'W') {      cellColor = TFT_WHITE; }
    else if (curPix == 'Y') { cellColor = TFT_YELLOW; }
    else if (curPix == 'B') { cellColor = TFT_BLUE; }
    else if (curPix == 'R') { cellColor = TFT_RED; }
    else if (curPix == 'G') { cellColor = 0x5E85; }
    if (curPix != 'Z' and scale == 1) {
      tft.drawPixel(x + i % imgW, y + i / imgW, cellColor);
    }
    else if (curPix != 'Z' and scale > 1) {
      tft.fillRect(x + scale*(i%imgW),y + 
         scale*(i/imgW),scale,scale,cellColor);
    }
  }
}
//=========================== button functions =====================
void up() {
  if (millis() - offsetT >= 50 and play) {
    changeShipX = 0; changeShipY = -6; offsetT = millis();
  }
}
//==================================================================
void down() {
  if (millis() - offsetT >= 50 and play) {
    changeShipX = 0; changeShipY = 6; offsetT = millis();
  }
}
//==================================================================
void left() {
  if (millis() - offsetT >= 50 and play) {
    changeShipX = -6; changeShipY = 0; offsetT = millis();
  }
}
//==================================================================
void right() {
  if (millis() - offsetT >= 50 and play) {
    changeShipX = 6; changeShipY = 0; offsetT = millis();
  }
}
//==================================================================
void select() {
  if (millis() - offsetF >= 500 and play) {
    fire = true; offsetF = millis();
  }
  if (!beginGame) { beginGame = true;}
}
//==================================================================

```


## 参考情報

色々なサンプルコードのURL:  
https://wiki.seeedstudio.com/Wio-Terminal-LCD-Fonts/   
wget https://raw.githubusercontent.com/Bodmer/TFT_eSPI/master/examples/320%20x%20240/Free_Font_Demo/Free_Fonts.h

https://wiki.seeedstudio.com/Wio-Terminal-Buttons/   
https://wiki.seeedstudio.com/Wio-Terminal-Switch/   
https://wiki.seeedstudio.com/Wio-Terminal-Buzzer/   
https://wiki.seeedstudio.com/Wio-Terminal-Infrared-Emitter/   

[Wio Terminalをはじめよう(ピン配置、データシート、回路図など)](https://wiki.seeedstudio.com/jp/Wio-Terminal-Getting-Started/)  

[Platform/Wio Terminal/Network/Overview](https://wiki.seeedstudio.com/Wio-Terminal-Network-Overview/)  
download url:   
https://files.seeedstudio.com/wiki/Wio-Terminal/res/ameba-image-Tool-v2.4.1.zip  
https://files.seeedstudio.com/wiki/Wio-Terminal/res/20200601-rtl8720d-images-v2.2.0.0.zip  

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

https://wiki.seeedstudio.com/Wio-Terminal-Wi-Fi/  

\# ESP8266ライブラリと互換性があるので以下が参考になる：   
https://arduino-esp8266.readthedocs.io/en/latest/esp8266wifi/readme.html  
https://arduino-esp8266.readthedocs.io/en/latest/esp8266wifi/udp-examples.html  

以上

