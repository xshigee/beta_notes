
2020/8/16:  
接続、スケッチ改版

2020/8/15:  
初版

PlatformIO XIAO WiFi-module
# PlatformIO XIAO WiFi-module

## 概要
XIAOに以下のWiFiモジュールを接続する(STARWARS)       
本記事は
「[XIAOを使ってWiFiモジュール(ESP-WROOM-02)の動作確認をする](https://beta-notes.way-nifty.com/blog/2020/07/post-6fd308.html)」の続きになる。   
(ホストPCとしてはubuntuを想定している)

・[ＥＳＰ－ＷＲＯＯＭ－０２開発ボード(AE-ESP-WROOM02-DEV)](http://akizukidenshi.com/catalog/g/gK-12236/)   
このボードには出荷時にATファームウェアが書き込まれている。(WiFiモジュールとして使用する場合、Arduinoのファームなどを書き込まない)

## 接続

ＥＳＰ－ＷＲＯＯＭ－０２開発ボード(AE-ESP-WROOM02-DEV)の場合
| AE-ESP-WROOM02-DEV | XIAO |
| ---: | :--- |
| 1(3V3入力)|　USB給電の場合、接続しない |
| 2(EN) | D8 |
| 14(TXD) | D7(RX) |
| 13(RXD) | D6(TX) |
| 9(GND) | GND |

USB給電の場合は、オンボードのUSBにホストＰＣなどを接続して行なう。（USBシリアルとしては使用しない）   
XIAOのUSBシリアルして使用するので、XIOAのUSBはホストPCに接続して通信ソフトを起動する。  


## starwars_test
telnetでASCIIアートの「StarWars」アニメを表示するプログラム：  
src/starwars_test.ino
```c++


// set your wifi ssid, passwd
#define MY_SSID "your_ssid"  
#define MY_PASSWD "your_passwd" 

#define esp8266_reset_pin 8 // Connect this pin to CH_PD(chip power-down) on the esp8266, not reset. 

char res[6000]; // need big buffer for StarWars

bool printReply = true;
const char Hline[] = "-----\n\r";
 
char ipAddress [20];
 
void getReply(int wait)
{ 
    int tempPos = 0;
    long int time = millis();
    while( (time + wait) > millis())
    { 
        while(Serial1.available()>0)
        {
            char c = Serial1.read(); 
            //if (tempPos < 500) { res[tempPos] = c; tempPos++;   }
            res[tempPos] = c; tempPos++;
        }
        res[tempPos] = 0;
    } 
 
    if (printReply) { Serial.println( res );  Serial.println(Hline);     }
}
void setup() 
{
  // **** we need chip rest ****
  pinMode(esp8266_reset_pin, OUTPUT);
  digitalWrite(esp8266_reset_pin, LOW);//Start with radio off
  //delay(500);
  delay(1000);
  digitalWrite(esp8266_reset_pin, HIGH); // select the radio
  // **** end of chip rest ****

  // serials init
  while (!Serial);
  while (!Serial1);
  Serial.begin(115200);  //serial port of Arduino Main
  Serial1.begin(115200);  //serial port of ESP8266
  delay(3000);
  //Serial.print("Type a key to start.\r\n");
  //while(!Serial.available()) continue; // wait for some input

  //pinMode(LED_BUILTIN, OUTPUT);

  //delay(3000);
 
  Serial.print("\r\n\r\n\r\n\r\n\r\n");

// no need rest command
/*********
  Serial1.print("AT+RST\r\n");
  getReply(2000);
***********/

  Serial1.print("AT+GMR\r\n");
  getReply(2000);

  Serial1.print("AT+CWQAP\r\n");
  getReply(2000);
 
  Serial1.print("AT+CWMODE=1\r\n");
  getReply(2000);

  // set your own wifi
  Serial1.printf("AT+CWJAP=\"%s\",", MY_SSID); 
  Serial1.printf("\"%s\"\r\n", MY_PASSWD); 
  getReply(10000);
 
  Serial1.print("AT+CIFSR\r\n");
  getReply(10000);

   //-- get IP Address -- 
   int len = strlen( res ); 
      bool done=false;
      bool error = false;
      int pos = 0;
      while (!done)
      {
           if ( res[pos] == '\"') { done = true;} 
           pos++;
           if (pos > len) { done = true;  error = true;}
      }
 
      if (!error)
      {
            int buffpos = 0;
            done = false;
            while (!done)
            {
               if ( res[pos] == '\"' ) { done = true; }
               else { ipAddress[buffpos] = res[pos];    buffpos++; pos++;   }
            }
            ipAddress[buffpos] = 0;
      }
      else { strcpy(ipAddress,"ERROR"); }
 
      Serial.printf("ipAddress:%s\r\n\r\n",ipAddress);

      Serial1.print("AT+CIPMUX=1\r\n");
      getReply( 1500 );
 
      Serial1.print("AT+CIPSERVER=1,80\r\n");
      getReply( 1500 );

      // display off
      printReply = false;

      // telnet access
      //Serial.print("AT+CIPSTART=1,\"TCP\",\"towel.blinkenlights.nl\",23\r\n");
      //Serial1.print("AT+CIPSTART=1,\"TCP\",\"towel.blinkenlights.nl\",23\r\n");
      // do this if DNS fail
      Serial.print("AT+CIPSTART=1,\"TCP\",\"94.142.241.111\",23\r\n");
      Serial1.print("AT+CIPSTART=1,\"TCP\",\"94.142.241.111\",23\r\n");

}


//-------------------------------------------------------

void get_res(void) {
  int cpos = 0; 
  for(int cc=0; cc<24000000; cc++) {
     while (Serial1.available()) {
       res[cpos] = (char)Serial1.read(); cpos++;
     };
  };
  res[cpos] = 0;
}

void get_res2(void) {
  int cpos = 0; 
  for(int cc=0; cc<500000; cc++) {
     while (Serial1.available()) {
       res[cpos] = (char)Serial1.read(); cpos++;
     };
  };
  res[cpos] = 0;
}

//-------------------------------------------------------

void dispatch(void)
{
  // setup string pointer table
  char *h[20]; for(int i=0; i<20; i++) h[i]=NULL;
  h[0] = &res[0];
  int pi = 1;
  int cp;

  int len = (int)strlen(res); // keep length of res[]
  for (int i=0; i<len; i++) {
     if ((res[i]=='+') && (res[i+1]=='I') && (res[i+2]=='P') && (res[i+3]=='D') ) { // check +IPD
        //for (cp = i+3; cp<len; cp++) {
        for (cp = i+4; cp<len; cp++) {
            if (res[cp]==':') break;
        }
        res[i] = 0; // make string terminator
        h[pi] = &res[cp+1]; pi++;
     }
  };
  // display removed IPD header
  for(int i=0; i<20; i++) {
    if (h[i] != NULL) Serial.print(h[i]);
  }
}
 
//------------

void loop()
{
  digitalWrite(LED_BUILTIN, HIGH); // debug: for indicate in-loop

/**
  // test loop
  while(true) {
    if (Serial1.available()) Serial.write((char)Serial1.read());
    if (Serial.available()) Serial1.write((char)Serial.read());
 };
**/

  while(true) {
     get_res2();
     //getReply(1000);
     if ((int)strlen(res) != 0) {
         //usart0_print("\r\nRES:\r\n");
         //usart0_println(res);
         dispatch();
     } else {
         //usart0_print("."); // indicating input wait


     }
  }

}
 ```
以下については、自分の環境に合わせて変更すること：  
#define MY_SSID "your_ssid"  
#define MY_PASSWD "your_passwd"   

書き込み後に「picocom /dev/ttyACM0 -b115200」で通信ソフトを起動すると以下のような出力が表示される：
```bash


$ picocom /dev/ttyACM0 -b115200

AT+GMR
AT version:1.7.4.0(May 11 2020 19:13:04)
SDK version:3.0.4(9532ceb)
compile time:May 27 2020 10:12:20
Bin version(Wroom 02):1.7.4
OK
WIFI GOT IP

-----

AT+CWQAP

OK
WIFI DISCONNECT

-----

AT+CWMODE=1

OK

-----

AT+CWJAP="YOUR_SSID","YOUR_PASSWD"
WIFI CONNECTED
WIFI GOT IP

OK

-----

AT+CIFSR
+CIFSR:STAIP,"192.168.0.19"
+CIFSR:STAMAC,"84:f3:eb:87:14:72"

OK

<省略>

# STARWARSのASCIIアートのアニメが表示される
```
実行するとStarWarsのASCIIアートのアニメが表示される。


## 参考情報


[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

[ESP8266をTCPサーバとTCPクライアントにするATコマンド実例 (1/4)](https://monoist.atmarkit.co.jp/mn/articles/1510/19/news007.html)  
[esp8266_at_command_examples_en.pdf](https://www.espressif.com/sites/default/files/documentation/4b-esp8266_at_command_examples_en.pdf)  

[ESP8266をWifiモデムとして使う - ATコマンドによるMQTT通信](http://nopnop2002.webcrow.jp/ESP-AT-Firmware/ESP-AT-Firmware-31.html)  
[MQTT_via_ESP01](https://github.com/nopnop2002/MQTT_via_ESP01)   
[Arduino WiFi library for ESP8266 modules](https://github.com/bportaluri/WiFiEsp)  

[espressif/ESP8266_AT - examples](https://github.com/espressif/ESP8266_AT/wiki/at_example_0020000903)   
[Espressif Documentation](https://www.espressif.com/en/support/documents/technical-documents)  

「wio lite RISC-V」による実装例：  
・[Wio_Lite_RISC-VボードでWiFiを動かす（その4：MQTT）](https://beta-notes.way-nifty.com/blog/2020/06/post-03f036.html)   
・[Wio_Lite_RISC-VボードでWiFiを動かす（その3：OSC）](https://beta-notes.way-nifty.com/blog/2020/06/post-639150.html)   
・[Wio_Lite_RISC-VボードでWiFiを動かす（その２：STARWARS,REST-API）](https://beta-notes.way-nifty.com/blog/2020/06/post-a531f3.html)  
・[Wio_Lite_RISC-VボードでWiFiを動かす](https://beta-notes.way-nifty.com/blog/2020/06/post-e3a206.html)  
・[開発ツールPlatformIOをcliで使う(Wio_Lite_RISC-V版)](https://beta-notes.way-nifty.com/blog/2020/06/post-f72459.html)

以上
