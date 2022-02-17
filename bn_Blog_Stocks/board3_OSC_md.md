
2020/7/22:  
スケッチ改版

2020/7/21:  
初版

board3 OSC
# board3 OSC

## 概要
Wio-Terminal/ESP8622/ESP32ボードを共通のスケッチで動かす(OSC編)    
(ホストPCとしてはubuntuを想定している)

## OSCcodecライブラリ
OSC_codecライブラリとしてプロジェクトのsrcディレクトリに以下の2つのファイルを置く：

src/OSCmsgCodec.h
```c++


// OSC message Codec Header
// 2013/10/28
#ifndef _OSCMSGCODEC_H
#define _OSCMSGCODEC_H

#include <string.h>

int encOSCmsg(char *packet , union OSCarg *msg);
// makes packet from OSC message and returns packet size

void decOSCmsg(char *packet , union OSCarg *msg); 
// makes OSC message from packet

union OSCarg {
  // char*, int and float are assumed four bytes
  char *address;
  char *typeTag;
  long int i; // int32 for Arduino(16bits)
  float f;
  char *s;
  struct {
    long int len; // is "int i"
    char *p;
  } 
  blob;
  char m[4];  // for MIDI
  char _b[4]; // endian conversion temp variable
};

#endif // _OSCMSGCODEC_H
```

src/OSCmsgCodec.cpp
```c++


/*
<----------------------------------------------------------------------------------

 OSC message Codec(encoder/decoder)
 
 version: 1.3 (2014/ 8/31) encoder bufix(enoceder returns byte length)   
 version: 1.2 (2014/ 8/30) decoder bufix 
 version: 1.1 (2013/11/20) support BIG_ENDIAN by #define
 version: 1.0 (2013/11/10)

 Copyright (C) 2011,2013,2014 S.Komatsu
 released under the MIT License: http://mbed.org/license/mit

 please refer to: http://opensoundcontrol.org/introduction-osc for OSC(Open Sound Control)

  The followings are supported:

    Features: 
     Packet Parsing (Client)
     Packet Construction (Server)
     Bundle NOT Support
     Timetag NOT Support

    Type Support: 
     i: int32
     b: blob
     s: string
     f: float32
     m: MIDI message(port id, status byte, data1, data2) // I don't know the detail
     
  Change Log:
   Bug(string length is not correct in encoding) Fix on 2013/11/10 (v0.9 -> v1.0)

>----------------------------------------------------------------------------------
*/

#include "OSCmsgCodec.h"
//#define BIG_ENDIAN

int lenAlign4B(int len) {
    if ((len % 4) == 0) {return len; }
    else {return len+4-(len % 4);}
}


int encOSCmsg(char *packet , union OSCarg *msg){
  // *** important notice ***
  // output buffer must be cleared before call this
  char *p, *s, *d, *typeTag; 
  char c;

  p=packet;
  d=p;
  s=msg[0].address; // address
  for(int i=0; i<strlen(msg[0].address); i++) *d++ = *s++;
  *d=0; // terminator
//  p += 4*((strlen(msg[0].address)+1)/4+1);
  p += lenAlign4B(strlen(msg[0].address)+1);
  // 
  s=msg[1].typeTag;
  d=p;
  for(int i=0; i<strlen(msg[1].typeTag); i++) *d++ = *s++; 
  *d=0; // terminator   
//  p += 4*((strlen(msg[1].s)+1)/4+1);
  p += lenAlign4B(strlen(msg[1].typeTag)+1);
  //
  typeTag=msg[1].s+1; // skip ','
  for(int n=0; n<strlen(typeTag); n++){
    c = typeTag[n];
    if (('s'==c)) {
      s=msg[n+2].s;
      d=p;
      for(int i=0; i<strlen(msg[n+2].s); i++) *d++ = *s++;
      *d=0; // terminater    
//     p += 4*((strlen(msg[n+2].s)+1)/4+1);
      p += lenAlign4B(strlen(msg[n+2].s)+1);
    } 
    else if (('i'==c)||('f'==c)) {
#ifdef BIG_ENDIAN
      // no change endian (big to big)
      p[0]=msg[n+2]._b[0];
      p[1]=msg[n+2]._b[1];
      p[2]=msg[n+2]._b[2];
      p[3]=msg[n+2]._b[3];
#else
      // change endian (little to big)
      p[0]=msg[n+2]._b[3];
      p[1]=msg[n+2]._b[2];
      p[2]=msg[n+2]._b[1];
      p[3]=msg[n+2]._b[0];
#endif
        p +=4;  
    } 
    else if ('b'==c) {
      // put length of blog
#ifdef BIG_ENDIAN
      // no change endian (big to big)
      p[0]=msg[n+2]._b[0];
      p[1]=msg[n+2]._b[1];
      p[2]=msg[n+2]._b[2];
      
      p[3]=msg[n+2]._b[3];
#else
      // change endian (little to big)
      p[0]=msg[n+2]._b[3];
      p[1]=msg[n+2]._b[2];
      p[2]=msg[n+2]._b[1];
      p[3]=msg[n+2]._b[0];
#endif
      p +=4;  
      // get ponter of blog (copy to msg[n].blog.p)
      s=msg[n+2].blob.p;
      d=p;
      for(int i=0; i<msg[n+2].blob.len; i++) *d++ = *s++;    
      p += 4*(msg[n+2].blob.len/4+1);       
    } 
    else if ('m'==c) {
      // get midi data (copy to msg[n].m[])
      p[0]=msg[n+2].m[0]; 
      p[1]=msg[n+2].m[1]; 
      p[2]=msg[n+2].m[2];
      p[3]=msg[n+2].m[3];
      p +=4;  
    } 
    else {
      //printf("*** Not Supported TypeTag:%s ****\n",typeTag);
    }
  };
  //return (p-packet); // return packet size
  // bugfix 2014/8/31
  return sizeof(char)*(p-packet);  // return byte length   
};
    
void decOSCmsg(char *packet , union OSCarg *msg){
  // Caution: the returned result points to packet as blobs or strings (not newly allocatd)
  char *p, *typeTag; 
  char c; int n;

  msg[0].address = packet; // address
  msg[1].typeTag = packet+4*((strlen(msg[0].s)+1)/4+1);//typeTag
  typeTag=msg[1].s+1; // skip ','
  
  // bugfix 2014/8/30
  if (strlen(typeTag)%2 == 0) p= msg[1].s+4*((strlen(msg[1].s)+1)/4); 
  else  p= msg[1].s+4*((strlen(msg[1].s)+1)/4+1);
  
  for(n=0; n<strlen(typeTag); n++){
    c = typeTag[n];
    if (('s'==c)) {
      msg[n+2].s=p;
      //p += 4*((strlen(msg[n+2].s)+1)/4+1);
      p += lenAlign4B(strlen(msg[n+2].s)+1);
    } 
    else if (('i'==c)||('f'==c)) {
#ifdef BIG_ENDIAN
      // no change endian (big to big)
      msg[n+2]._b[0]=p[0];
      msg[n+2]._b[1]=p[1];
      msg[n+2]._b[2]=p[2];
      msg[n+2]._b[3]=p[3];
#else
      // change endian (big to little)
      msg[n+2]._b[3]=p[0];
      msg[n+2]._b[2]=p[1];
      msg[n+2]._b[1]=p[2];
      msg[n+2]._b[0]=p[3];
#endif
      p +=4;  
    } 
    else if ('b'==c) {
      // get lenth of blog (copy to msg[n].blog.len)
#ifdef BIG_ENDIAN
      // no change endian (big to big)
      msg[n+2]._b[0]=p[0];
      msg[n+2]._b[1]=p[1];
      msg[n+2]._b[2]=p[2];
      msg[n+2]._b[3]=p[3];
#else
      // change endian (big to little)
      msg[n+2]._b[3]=p[0];
      msg[n+2]._b[2]=p[1];
      msg[n+2]._b[1]=p[2];
      msg[n+2]._b[0]=p[3];
#endif
      p +=4;
      // get ponter of blog (copy to msg[n].blog.p)
      msg[n+2].blob.p=p;
      //p += 4*(msg[n+2].blob.len/4+1);       
      p += lenAlign4B(msg[n+2].blob.len+1);
    } 
    else if ('m'==c) {
      // get midi data (copy to msg[n].m[])
      msg[n+2].m[0]=p[0]; 
      msg[n+2].m[1]=p[1]; 
      msg[n+2].m[2]=p[2];
      msg[n+2].m[3]=p[3];
      p +=4;  
    } 
    else {
      //printf("*** Not Supported TypeTag:%s ****\n",typeTag);
    }
 };
};
```

## デモ・スケッチ

src/OSC_test.ino
```c++

// select board
////#define WIO_TERMINAL
////#define ESP8266
////#define ESP32
/*
    OSC sender/receiver
    Forked for Wio-Terminal/ESP8266(ESP-WROOM-02)/ESP32
    from: This sketch sends random data over UDP on a ESP32 device

*/
#ifdef WIO_TERMINAL
#include <AtWiFi.h>
#endif
#ifdef ESP8266
#include <ESP8266WiFi.h>
#endif
#ifdef ESP32
#include <WiFi.h>
#endif
#include <WiFiUdp.h>

// WiFi network name and password:
const char* ssid = "your_ssid";
const char* passwd = "your_passwd";

//IP address to send UDP data to:
const char* udpAddress = "192.168.0.21"; // target IP
const int udpInPort = 8000; // incomming port
const int udpOutPort = 9000; // outgoing port

//<<<<<<<<<<<<<<<<<<<<<
// OSC related
#include "OSCmsgCodec.h"

#define PACKET_SIZE 512
#define RECBUF_SIZE 256
 
union OSCarg msg[10], outmsg[10];
char buff[PACKET_SIZE],packet[PACKET_SIZE];
//>>>>>>>>>>>>>>>>>>>>
char recbuf[128];  // buffer for incoming packets

//The udp library class
WiFiUDP udp;

void setup()
{
  Serial.begin(115200);
  Serial.println("OSC test program start...");
  Serial.println("");

  // delete old config
  WiFi.disconnect(true);
  Serial.printf("Connecting to %s ", ssid);
  WiFi.begin(ssid, passwd);
  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }
  Serial.println(" connected");

  udp.begin(udpInPort);
  Serial.printf("Now listening at IP %s, UDP port %d\r\n", 
                WiFi.localIP().toString().c_str(), udpInPort);
}

void loop() {
  int packetSize = udp.parsePacket();
  if (packetSize>0)
  {
    Serial.printf("Received %d bytes from %s, port %d\r\n", 
       packetSize, udp.remoteIP().toString().c_str(), udpInPort);
    int len = udp.read(recbuf, 128);
    if (len > 0) {
        decOSCmsg(recbuf, msg);
        Serial.printf("incomming OSC msg:\r\n%s %s", msg[0].address, msg[1].typeTag);
        for(int m=0; m < (strlen(msg[1].typeTag)-1); m++) {
            if (msg[1].typeTag[m+1]=='f') Serial.printf(" %f",msg[m+2].f);
            if (msg[1].typeTag[m+1]=='i') Serial.printf(" %d",msg[m+2].i);                
        };
        Serial.printf("\r\n");
        Serial.printf("-------------------\r\n");
    }
  } else {
    // test send
    // fader
    // send OSC message with random values (to touchOSC)
    // make OSC message for sending
    outmsg[0].address="/1/fader5";    
    outmsg[1].typeTag=",f";
    outmsg[2].f= rand()%1000/1000.0;
    memset(packet,0,sizeof(packet)); // clear send buff for OSC msg
    int plen=encOSCmsg(packet,outmsg);
    // send it 
    // Send a packet
    udp.beginPacket(udpAddress, udpOutPort);
    udp.write((const uint8_t*)packet, plen);
    udp.endPacket();
    //
    //Serial.print("."); // indicate sending packet
    //Wait
    delay(10); // seems 'must' (for platformio)
  }
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

// WiFi network name and password:
const char* ssid = "your_ssid";
const char* passwd = "your_passwd";

//IP address to send UDP data to:
const char* udpAddress = "192.168.0.15"; // target IP
const int udpInPort = 8000; // incomming port
const int udpOutPort = 9000; // outgoing port
```
udpAddressは、対向するOSC受信デバイス(touchOSCなど)のIPを設定する。

書き込み後に「picocom /dev/ttyACM0 -b115200」または「picocom /dev/ttyUSB0 -b115200」で通信ソフトを起動すると以下のような出力が表示される：
```

$ picocom /dev/ttyACM0 -b115200

........... connected
Now listening at IP 192.168.0.11, UDP port 8000

```
上の例ではTouchOSCから192.168.0.11:8000へOSCパッケットに送る(ようにTouchOSCを設定する)。

または、動作としては、touchOSCのsimpleレイアウトの画面で、最上行の受信アイコンがOSCを受信すると「赤」になり、fader5のfaderが送られてきたOSCの内容に従ってランダムに変化する。

## Arduino-IDEのボードマネージャーのURL
以下のURLを設定することで３つのボード(Wio-Terminal/ESP8266/ESP32)が使用可能になる。
```

https://files.seeedstudio.com/arduino/package_seeeduino_boards_index.json
http://arduino.esp8266.com/stable/package_esp8266com_index.json
https://dl.espressif.com/dl/package_esp32_index.json
```

## 問題点
Wio-Terminalでは、OSC受信は、動作しなかった。（原因不明）

## 参考情報

[TouchOSC - Modular touch control surface for OSC & MIDI](https://hexler.net/products/touchosc)  
[TouchOSC - iPhone](https://apps.apple.com/app/touchosc/id288120394)  
[TouchOSC - Android](https://play.google.com/store/apps/details?id=net.hexler.touchosc_a) 

OSCmsgCode lib:   
https://os.mbed.com/users/xshige/code/Sparkfun_CC3000_WiFi_OSCtranceiver//raw-file/e62251d890c1/OSCmsgCodec.h  
https://os.mbed.com/users/xshige/code/Sparkfun_CC3000_WiFi_OSCtranceiver//raw-file/e62251d890c1/OSCmsgCodec.cpp  
https://os.mbed.com/users/xshige/code/Sparkfun_CC3000_WiFi_OSCtranceiver//file/e62251d890c1/main.cpp/   

[Platform/Wio Terminal Network/Wi-Fi - example sketches](https://wiki.seeedstudio.com/Wio-Terminal-Wi-Fi/)  

[Wio Terminalをはじめよう(ピン配置、データシート、回路図など)](https://wiki.seeedstudio.com/jp/Wio-Terminal-Getting-Started/)  

[Platform/Wio Terminal/Network/Overview](https://wiki.seeedstudio.com/Wio-Terminal-Network-Overview/)  
download url:   
https://files.seeedstudio.com/wiki/Wio-Terminal/res/ameba-image-Tool-v2.4.1.zip  
https://files.seeedstudio.com/wiki/Wio-Terminal/res/20200601-rtl8720d-images-v2.2.0.0.zip  

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

以上
