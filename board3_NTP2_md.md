
2020/8/1:  
初版

board3 NTP Client 2
# board3 NTP Client 2

## 概要
Wio-Terminal/ESP8622/ESP32ボードを共通のスケッチで動かす(NTP-CLIENT編)    
(ホストPCとしてはubuntuを想定している)


## 外部ライブラリ(platformioの場合)
以下を実行してライブラリをインストールする：
```bash

# NTP Client lib インストール
#pio lib install "NTPClient"
pio lib install 551

```

## デモ・スケッチ
  
src/NTP-Client2.ino
```c++

// select board
////#define WIO_TERMINAL
////#define ESP8266
////#define ESP32
//------------------

// NTP client2 for Wio-Terminal/ESP8266/ESP32

#ifdef WIO_TERMINAL
#include <AtWiFi.h>
#endif
#ifdef ESP8266
#include <ESP8266WiFi.h>
#endif
#ifdef ESP32
#include <WiFi.h>
#endif

#include <NTPClient.h>
#include <WiFiUdp.h>

const char *ssid     = "your_ssid";
const char *password = "your_passwd";

//<----------------------------------------------------------
// derived/forked from http://mrkk.ciao.jp/memorandom/unixtime/unixtime.html
#define ARRAYSIZE(_arr) (sizeof(_arr) / sizeof(_arr[0]))
#define TIME_OFFSET 0
#define SECONDS_IN_A_DAY    (24*60*60)
#define EPOCH_DAY       (1969*365L + 1969/4 - 1969/100 + 1969/400 + 306)    //  days from 0000/03/01 to 1970/01/01
#define UNIX_EPOCH_DAY  EPOCH_DAY
#define YEAR_ONE        365
#define YEAR_FOUR       (YEAR_ONE * 4 + 1)  //  it is YEAR_ONE*4+1 so the maximum reminder of day / YEAR_FOUR is YEAR_ONE * 4 and it occurs only on 2/29
#define YEAR_100        (YEAR_FOUR * 25 - 1)
#define YEAR_400        (YEAR_100*4 + 1)    //  it is YEAR_100*4+1 so the maximum reminder of day / YEAR_400 is YEAR_100 * 4 and it occurs only on 2/29

void ConvertUnixTimeToLocalTime(uint64_t unixtime, uint32_t *pyear, uint8_t *pmonth, uint8_t *pday, uint8_t *phour, uint8_t *pminute, uint8_t *psecond) {
    uint32_t unixday;
    uint16_t year = 0;
    uint8_t  leap = 0;
    uint32_t n;
    uint8_t month, day, weekday;
    uint8_t hour, minute, second;
    static const uint16_t monthday[] = { 0,31,61,92,122,153,184,214,245,275,306,337 };

    unixtime += TIME_OFFSET;
    second = unixtime % 60;
    minute = (unixtime / 60) % 60;
    hour = (unixtime / 3600) % 24;

    unixday = (uint32_t)(unixtime / SECONDS_IN_A_DAY);
    weekday = (uint8_t)((unixday + 3) % 7); //  because the unix epoch day is thursday
    unixday += UNIX_EPOCH_DAY;  //  days from 0000/03/01 to 1970/01/01

    year += 400 * (unixday / YEAR_400);
    unixday %= YEAR_400;

    n = unixday / YEAR_100;
    year += n * 100;
    unixday %= YEAR_100;

    if (n == 4){
        leap = 1;
    } else {
        year += 4 * (unixday / YEAR_FOUR);
        unixday %= YEAR_FOUR;
        
        n = unixday / YEAR_ONE;
        year += n;
        unixday %= YEAR_ONE;
        if (n == 4) {
            leap = 1;
        }
    }
    if (leap != 0) {
        month = 2;
        day = 29;
    }
    else {
        month = (unixday * 5 + 2) / 153;
        day = unixday - monthday[month] + 1;    //  
        month += 3;
        if (month > 12) {
            ++year;
            month -= 12;
        }
    }
    *psecond = second;
    *pminute = minute;
    *phour = hour;
    *pyear = year;
    *pmonth = month;
    *pday = day;
}
//>----------------------------------------------------------

char *weekday[] = {"Sun","Mon","Tue","Wed","Thu","Fri","Sat"};
uint32_t year;
uint8_t month, day, hour, minu, sec;

WiFiUDP ntpUDP;
NTPClient timeClient(ntpUDP, "ntp.nict.jp", 3600*9, 60000); // set JST
// You can specify the time server pool and the offset, (in seconds)
// additionaly you can specify the update interval (in milliseconds).
// NTPClient timeClient(ntpUDP, "europe.pool.ntp.org", 3600, 60000);

void setup(){
  Serial.begin(115200);
  WiFi.begin(ssid, password);

  while ( WiFi.status() != WL_CONNECTED ) {
    delay ( 500 );
    Serial.print ( "." );
  }

  timeClient.begin();
}

void loop() {
  timeClient.update();

  Serial.println(timeClient.getFormattedTime());
  Serial.println(weekday[timeClient.getDay()]); // Sun, Mon, .., Sat
  Serial.println(timeClient.getHours());
  Serial.println(timeClient.getMinutes());
  Serial.println(timeClient.getSeconds());
  Serial.println(timeClient.getEpochTime()); // unsigned long getEpochTime()

  ConvertUnixTimeToLocalTime(timeClient.getEpochTime(),
    &year, &month, &day, &hour, &minu, &sec);
  Serial.printf("%4d/%02d/%02d: %02d:%02d:%02d\r\n",year, month, day, hour, minu, sec);
  Serial.println();

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

const char *ssid     = "your_ssid";
const char *password = "your_passwd";
```

書き込み後に「picocom /dev/ttyACM0 -b115200」または「picocom /dev/ttyUSB0 -b115200」で通信ソフトを起動すると以下のような出力が表示される：
```bash


$ picocom /dev/ttyACM0 -b115200


13:24:37
Sat
13
24
37
1596288277
2020/08/01: 13:24:37

13:24:38
Sat
13
24
38
1596288278
2020/08/01: 13:24:38

13:24:39
Sat
13
24
39
1596288279
2020/08/01: 13:24:39
```

## Arduino-IDEのボードマネージャーのURL
以下のURLを設定することで３つのボード(Wio-Terminal/ESP8266/ESP32)が使用可能になる。
```

https://files.seeedstudio.com/arduino/package_seeeduino_boards_index.json
http://arduino.esp8266.com/stable/package_esp8266com_index.json
https://dl.espressif.com/dl/package_esp32_index.json
```

## デモ・スケッチ２(参考)
Wio-Terminalの場合、必要な関数が実装されていないのでコンパイルエラーになるが
ESP8266/ESP32は正常にコンパイル&実行ができる。
  
src/NTP-Client.ino
```c++

// select board
////#define WIO_TERMINAL
////#define ESP8266
////#define ESP32
//------------------

// NTP Client for Wio-Terminal/ESP8266/ESP32

#ifdef WIO_TERMAL
#include <AtWiFi.h>
#endif
#ifdef ESP8266
#include <ESP8266WiFi.h>
#endif
#ifdef ESP32
#include <WiFi.h>
#endif

#include <time.h>

#define WIFI_SSID   "your_ssid"
#define WIFI_PASSWORD   "your_passwd"

void setup() {
  Serial.begin(115200);
  delay(100);
  Serial.print("\r\n\nReset:\r\n");

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  while(WiFi.status() != WL_CONNECTED) {
    Serial.print('.');
    delay(500);
  }
  Serial.println();
  Serial.printf("Connected, IP address: ");
  Serial.println(WiFi.localIP());

  configTzTime("JST-9", "ntp.nict.jp", "ntp.jst.mfeed.ad.jp");   // 2.7.0以降, esp32コンパチ
}

void loop() {
  time_t t;
  struct tm *tm;
  static const char *wd[7] = {"Sun","Mon","Tue","Wed","Thr","Fri","Sat"};

  t = time(NULL);
  tm = localtime(&t);
/*****
  Serial.printf("ESP8266/Arduino ver%s :  %04d/%02d/%02d(%s) %02d:%02d:%02d\n",
        __STR(ARDUINO_ESP8266_GIT_DESC),
        tm->tm_year+1900, tm->tm_mon+1, tm->tm_mday,
        wd[tm->tm_wday],
        tm->tm_hour, tm->tm_min, tm->tm_sec);
****/
  Serial.printf("Arduino NTP:  %04d/%02d/%02d(%s) %02d:%02d:%02d\r\n",
        tm->tm_year+1900, tm->tm_mon+1, tm->tm_mday,
        wd[tm->tm_wday],
        tm->tm_hour, tm->tm_min, tm->tm_sec);
  delay(1000);
}
```


## 参考情報

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

以上
