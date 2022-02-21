
2020/6/23:  
初版

PlatformIO Wio Lite RISC-V WiFi3
# PlatformIO Wio Lite RISC-V WiFi3

## 概要
Wio_Lite_RISC-VボードでWiFiを動かす（その3：OSC）。
これは[「Wio_Lite_RISC-VボードでWiFiを動かす（その２）」](https://beta-notes.way-nifty.com/blog/2020/06/post-a531f3.html)の続きになる。ここでは、OSC(Open Sound Control)のデモプログラムについて記載する    
開発ツール(PlatformIOなど)のインストールについては[「開発ツールPlatformIOをcliで使う(Wio_Lite_RISC-V版)」](https://beta-notes.way-nifty.com/blog/2020/06/post-f72459.html)を参照のこと、ここでは、WiFiを動かすためのプログラムについて記載する。
(ホストPCとしてはubuntuを想定している)

## 接続
wio-lite-rvボードのPA9(TX). PA10(RX)、GNDをUSBシリアルに接続する。   
ちなみに、XIAOをUSBシリアルとして使用する場合は以下のように接続する：  
| wio-lite-rv | XIAO |
| ---: | :--- |
| PA10(RX) | D6(TX) |
| PA9(TX) | D7(RX) |
| GND | GND |

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

## OSC_recv_test
OSC受信プログラム：  
src/OSC_recv_test.cpp
```c++

#include <Arduino.h>
#include <stdio.h>
#include <stdarg.h>
#include <math.h>

// OSC related
#include "OSCmsgCodec.h"

#define PACKET_SIZE 512
#define RECBUF_SIZE 256
 
union OSCarg msg[10], outmsg[10];
char buff[PACKET_SIZE],packet[PACKET_SIZE];

#define MY_SSID "your_ssid"
#define MY_PASSWD "your_passwd"

#define TARGET_IP "192.168.0.23"
#define OUT_PORT 9000
#define IN_PORT  8000

char ipAddress [20];

static void wio_serial_init(void)
{
    // enable GPIO clock for USART0
    rcu_periph_clock_enable(RCU_GPIOA);
    // enable USART clock 
    rcu_periph_clock_enable(RCU_USART0);  
    /* connect port to USARTx_Tx(PA9) */ 
    gpio_init(GPIOA, GPIO_MODE_AF_PP, GPIO_OSPEED_50MHZ, GPIO_PIN_9);
    /* connect port to USARTx_Rx(PA10) */
    gpio_init(GPIOA, GPIO_MODE_IN_FLOATING, GPIO_OSPEED_50MHZ, GPIO_PIN_10);

    // enable GPIO clock for USART1
    rcu_periph_clock_enable(RCU_GPIOA);
    // enable USART clock 
    rcu_periph_clock_enable(RCU_USART1); 
    /* connect port to USARTx_Tx(PA2) */ 
    gpio_init(GPIOA, GPIO_MODE_AF_PP, GPIO_OSPEED_50MHZ, GPIO_PIN_2);
    /* connect port to USARTx_Rx(PA3) */
    gpio_init(GPIOA, GPIO_MODE_IN_FLOATING, GPIO_OSPEED_50MHZ, GPIO_PIN_3);

    // USART0 configure 
    usart_deinit(USART0);
       
    usart_baudrate_set(USART0, 115200U);
    usart_word_length_set(USART0, USART_WL_8BIT);
    usart_stop_bit_set(USART0, USART_STB_1BIT);
    usart_parity_config(USART0, USART_PM_NONE);
    usart_hardware_flow_rts_config(USART0, USART_RTS_DISABLE);
    usart_hardware_flow_cts_config(USART0, USART_CTS_DISABLE);
    usart_receive_config(USART0, USART_RECEIVE_ENABLE);
    usart_transmit_config(USART0, USART_TRANSMIT_ENABLE);
    usart_enable(USART0);

    // USART1 configure 
    usart_deinit(USART1);

    usart_baudrate_set(USART1, 115200U);
    usart_word_length_set(USART1, USART_WL_8BIT);
    usart_stop_bit_set(USART1, USART_STB_1BIT);
    usart_parity_config(USART1, USART_PM_NONE);
    usart_hardware_flow_rts_config(USART1, USART_RTS_DISABLE);
    usart_hardware_flow_cts_config(USART1, USART_CTS_DISABLE);
    usart_receive_config(USART1, USART_RECEIVE_ENABLE);
    usart_transmit_config(USART1, USART_TRANSMIT_ENABLE);
    usart_enable(USART1);

}

int _get0_char(void) {
    while ( usart_flag_get(USART0, USART_FLAG_RBNE)== RESET){
    }
   return (uint8_t)usart_data_receive(USART0);
}

int _get1_char(void) {
    while ( usart_flag_get(USART1, USART_FLAG_RBNE)== RESET){
    }
   return (uint8_t)usart_data_receive(USART1);
}

int _put0_char(int ch)
{
    usart_data_transmit(USART0, (uint8_t) ch );
    while ( usart_flag_get(USART0, USART_FLAG_TBE)== RESET){
    }
 
    return ch;
}

int _put1_char(int ch)
{
    usart_data_transmit(USART1, (uint8_t) ch );
    while ( usart_flag_get(USART1, USART_FLAG_TBE)== RESET){
    }
 
    return ch;
}

void usart0_printf(const char *fmt, ...) {
    char buf[100];
    va_list args;
    va_start(args, fmt);
    vsprintf(buf, fmt, args);
    va_end(args);
 
    char *p = buf;
    while( *p != '\0' ) {
        _put0_char(*p);
        p++;
    }
}
void usart0_print(const char *s) {
    while( *s != '\0' ) {
        _put0_char(*s);
        s++;
    }
}

void usart0_println(const char *s) {
    while( *s != '\0' ) {
        _put0_char(*s);
        s++;
    }
    _put0_char('\r');
    _put0_char('\n');
}

void usart1_printf(const char *fmt, ...) {
    char buf[100];
    va_list args;
    va_start(args, fmt);
    vsprintf(buf, fmt, args);
    va_end(args);
 
    char *p = buf;
    while( *p != '\0' ) {
        _put1_char(*p);
        p++;
    }
}
void usart1_print(const char *s) {
    while( *s != '\0' ) {
        _put1_char(*s);
        s++;
    }
}

void usart1_println(const char *s) {
    while( *s != '\0' ) {
        _put1_char(*s);
        s++;
    }
    _put1_char('\r');
    _put1_char('\n');
}


int usart1_available(void) {
  return ( usart_flag_get(USART1, USART_FLAG_RBNE)== SET);
}


//-------------------------------------------------------

char res[500];
void get_res(void) {
  int cpos = 0; 
  for(int cc=0; cc<24000000; cc++) {
     while ( usart_flag_get(USART1, USART_FLAG_RBNE)== SET) {
       //res[cpos] = (char) _get1_char(); cpos++;
       res[cpos] = (char) usart_data_receive(USART1); cpos++;
     };
  };
  res[cpos] = 0;
}
void get_res2(void) {
  int cpos = 0; 
  for(int cc=0; cc<500000; cc++) {
     while ( usart_flag_get(USART1, USART_FLAG_RBNE)== SET) {
       //res[cpos] = (char) _get1_char(); cpos++;
       res[cpos] = (char) usart_data_receive(USART1); cpos++;
     };
  };
  res[cpos] = 0;
}

void setup() 
{
  wio_serial_init();
  //Serial.begin(115200);  //serial port of GD32
  //Serial1.begin(115200);  //serial port of ESP8266
  pinMode(LED_BUILTIN, OUTPUT);
 
  delay(3000);

  usart0_print("\r\n\r\n\r\n\r\n\r\n");

  // rest ESP8266
  usart0_println("SEND:");
  usart0_print("AT+RST\r\n");
  usart1_print("AT+RST\r\n");
  get_res();
  usart0_println("RES:");
  usart0_println(res);
  usart0_println("-----------");

  // get firmware version 
  usart0_println("SEND:");
  usart0_print("AT+GMR\r\n");
  usart1_print("AT+GMR\r\n");
  get_res();
  usart0_println("RES:");
  usart0_println(res);
  usart0_println("-----------");
 
  usart0_println("SEND:");
  usart0_print("AT+CWQAP\r\n");
  usart1_print("AT+CWQAP\r\n");
  get_res();
  usart0_println("RES:");
  usart0_println(res);
  usart0_println("-----------");

  usart0_println("SEND:");
  usart0_print("AT+CWMODE=1\r\n");
  usart1_print("AT+CWMODE=1\r\n");
  get_res();
  usart0_println("RES:");
  usart0_println(res);
  usart0_println("-----------");

  //Serial1.println("AT+CWJAP=\"Your WiFi SSID\",\"Password\""); // add your own wifi
  usart0_println("SEND:"); 
  usart0_print("AT+CWJAP=\"");
  usart0_print(MY_SSID);
  usart0_print("\",\"");
  usart0_print(MY_PASSWD);
  usart0_print("\"\r\n");

  usart1_print("AT+CWJAP=\"");
  usart1_print(MY_SSID);
  usart1_print("\",\"");
  usart1_print(MY_PASSWD);
  usart1_print("\"\r\n");
  get_res();
  usart0_println("RES:");
  usart0_println(res);
  usart0_println("-----------");

  usart0_println("SEND:"); 
  usart0_print("AT+CIFSR\r\n");
  usart1_print("AT+CIFSR\r\n");
  get_res();
  usart0_println("RES:");

  usart0_println(res);
  usart0_println("-----------");

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
               else { ipAddress[buffpos] = res[pos]; buffpos++; pos++;}
            }
            ipAddress[buffpos] = 0;
      }
      else { strcpy(ipAddress,"ERROR"); }

      //usart0_printf("ipAddress:%s\r\n",ipAddress);
//-----------------
 
  usart0_println("SEND:"); 
  usart0_print("AT+CIPMUX=1\r\n"); 
  usart1_print("AT+CIPMUX=1\r\n"); 
  get_res();
  usart0_println("RES:");
  usart0_println(res);
  usart0_println("-----------");

  usart0_println("SEND:");
  usart0_print("AT+CIPSERVER=1,80\r\n"); 
  usart1_print("AT+CIPSERVER=1,80\r\n"); 
  get_res();
  usart0_println("RES:");
  usart0_println(res);
  usart0_println("-----------");

  usart0_printf("\r\nipAddress: %s\r\n\r\n",ipAddress); 

  // UDP access
  // for UDP
  usart0_printf("AT+CIPSTART=1,\"UDP\",\"%s\",%d,%d,0\r\n",TARGET_IP,OUT_PORT,IN_PORT);
  usart1_printf("AT+CIPSTART=1,\"UDP\",\"%s\",%d,%d,0\r\n",TARGET_IP,OUT_PORT,IN_PORT);
  get_res();
  usart0_println("RES:");
  usart0_println(res);
  usart0_println("-----------");
}

//------------

void dispatch(void)
{
  int datlen;
  int cp1,cp2,cp3;
  int len = (int)strlen(res); // keep length of res[]
  for (int i=0; i<len; i++) {
     if ((res[i]=='+') && (res[i+1]=='I') && (res[i+2]=='P') && (res[i+3]=='D') ) { // check +IPD
        // skip 1st ','
        for (cp1 = i+4; cp1<len; cp1++) { 
            if (res[cp1]==',') break;
        }
        // find 2nd ','
        for (cp2 = cp1+1; cp2<len; cp2++) { 
            if (res[cp2]==',') break;
        }
        char *dp = &res[cp2+1];
        // find ':'
        for (cp3 = cp2+1; cp3<len; cp3++) { 
            if (res[cp3]==':') break;
        }
        res[cp3]=0; // make string terminator
        datlen = atoi(dp);
        int d = 0;
        for (int s=cp3+1; s<(cp3+1+datlen); s++) {
          packet[d] = res[s]; d++;
        }; 
        packet[d]=0;
        //usart0_printf("datalen: %d\r\n",datlen);
        //
        // decode OSC messeage
        decOSCmsg(packet, msg);
        usart0_printf("incomming OSC msg:\r\n%s %s\r\n", msg[0].address, msg[1].typeTag);
        for(int m=0; m < ((int)strlen(msg[1].typeTag)-1); m++) {
             if (msg[1].typeTag[m+1]=='f') usart0_printf(" %d/1000",
                  (int)floor(1000*msg[m+2].f)); // %f not supported, replace with %d 
             if (msg[1].typeTag[m+1]=='i') usart0_printf(" %d",msg[m+2].i);                
        }
        usart0_printf("\r\n");
        usart0_printf("Received packet from: %s \r\n", TARGET_IP);
        usart0_printf("-------------------\r\n");
     }
  };

}
 
//------------

void loop()
{
  digitalWrite(LED_BUILTIN, HIGH); // debug: for indicate in-loop

/**
  // test loop
  while(true) {
    if ( usart_flag_get(USART1, USART_FLAG_RBNE)== SET) _put0_char(_get1_char());
    if ( usart_flag_get(USART0, USART_FLAG_RBNE)== SET) _put1_char(_get0_char());
 };
**/

  while(true) {
     get_res2();
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
```
#define MY_SSID "your_ssid"  
#define MY_PASSWD "your_passwd"   

#define TARGET_IP "192.168.0.23"
#define OUT_PORT 9000
#define IN_PORT  8000
```
TARGET_IPは、対向するOSC送信デバイス(touchOSCなど)のIPを設定する。

書き込み後に「picocom /dev/ttyACM0 -b115200」で通信ソフトを起動すると以下のような出力が表示される：
```bash


$ picocom /dev/ttyACM0 -b115200

SEND:
AT+GMR
RES:
AT+GMR
AT version:2.1.0.0-dev(d25f6d2 - Oct 15 2019 12:03:04)
SDK version:v3.2-192-g81655f39
compile time(525fbfe):Oct 17 2019 05:39:13
Bin version:2.0.0(WROOM-02)

...
<省略>
...

-----------

ipAddress: 192.168.0.26


# この時点でtouchOSCのfaderなどを動かす。
# (この例では、Wio-LiteのIPの"192.168.0.26"へOSCメッセージを送る)
# 以下のように受信したOSCメッセージが表示される：


incomming OSC msg:
/1/fader2 ,f
 983/1000
Received packet from: 192.168.0.23 
-------------------
incomming OSC msg:
/1/fader2 ,f
 670/1000
Received packet from: 192.168.0.23 
-------------------
incomming OSC msg:
/1/fader2 ,f
 554/1000
Received packet from: 192.168.0.23 
-------------------
incomming OSC msg:
/1/fader2 ,f
 776/1000
Received packet from: 192.168.0.23 
-------------------
incomming OSC msg:
/1/fader2 ,f
 700/1000
Received packet from: 192.168.0.23 
-------------------
incomming OSC msg:
/1/fader2 ,f
 837/1000
Received packet from: 192.168.0.23 
-------------------
incomming OSC msg:
/1/fader2 ,f
 575/1000
Received packet from: 192.168.0.23 
-------------------
incomming OSC msg:
/1/fader2 ,f
 814/1000
Received packet from: 192.168.0.23 
-------------------

```
printfの%fがサポートされていないので、整数の%dで代用している。(例：「814/1000」は、0.814を意味する)

## OSC_send_test
OSC送信プログラム：   
OSC受信プログラムのloop()関数を以下のものに置き換える：
```c++

void loop()
{
  digitalWrite(LED_BUILTIN, HIGH); // debug: for indicate in-loop

/**
  // test loop
  while(true) {
    if ( usart_flag_get(USART1, USART_FLAG_RBNE)== SET) _put0_char(_get1_char());
    if ( usart_flag_get(USART0, USART_FLAG_RBNE)== SET) _put1_char(_get0_char());
 };
**/

  while(true){
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
    ////usart0_printf("send: AT+CIPSEND=1,%d\r\n",plen);
    usart1_printf("AT+CIPSEND=1,%d\r\n",plen);
    delay(10);
    get_res2();
    for(int i=0; i<plen; i++) _put1_char(packet[i]);
    delay(10);
    get_res2();
    ////usart0_printf("res: %s\r\n",res);
    usart0_print("."); // indicating OSC sending
  }

}
```
以下については、OSC受信でもプログラムと同様に自分の環境に合わせて変更すること：  
```
#define MY_SSID "your_ssid"  
#define MY_PASSWD "your_passwd"   

#define TARGET_IP "192.168.0.14"
#define OUT_PORT 9000
#define IN_PORT  8000
```
TARGET_IPは、対向するOSC受信デバイス(touchOSCなど)のIPを設定する。

書き込み後に「picocom /dev/ttyACM0 -b115200」で通信ソフトを起動すると以下のような出力が表示される：

```

$ picocom /dev/ttyACM0 -b115200


SEND:
AT+RST
RES:
AT+RST

OK
I (1337827) wifi: state: 5 -> 0 (0)
WIFI DISCONNECT
rl
-----------
SEND:
AT+GMR
RES:
AT+GMR
AT version:2.1.0.0-dev(d25f6d2 - Oct 15 2019 12:03:04)
SDK version:v3.2-192-g81655f39
compile time(525fbfe):Oct 17 2019 05:39:13
Bin version:2.0.0(WROOM-02)

OK

...

<省略>
...

-----------

ipAddress: 192.168.0.28

AT+CIPSTART=1,"UDP","192.168.0.14",9000,8000,0
RES:
AT+CIPSTART=1,"UDP","192.168.0.14",9000,8000,0
local port:8000
change type:0
1,CONNECT

OK

-----------
#以下、OSC送信のたびに「.」が出力される。
.........................

```

動作としては、touchOSCのsimpleレイアウトの画面で、最上行の受信アイコンがOSCを受信すると「赤」になり、fader5のfaderが送られてきたOSCの内容に従ってランダムに変化する。


## 参考情報

[TouchOSC - Modular touch control surface for OSC & MIDI](https://hexler.net/products/touchosc)  
[TouchOSC - iPhone](https://apps.apple.com/app/touchosc/id288120394)  
[TouchOSC - Android](https://play.google.com/store/apps/details?id=net.hexler.touchosc_a) 

OSCmsgCode lib:   
https://os.mbed.com/users/xshige/code/Sparkfun_CC3000_WiFi_OSCtranceiver//raw-file/e62251d890c1/OSCmsgCodec.h  
https://os.mbed.com/users/xshige/code/Sparkfun_CC3000_WiFi_OSCtranceiver//raw-file/e62251d890c1/OSCmsgCodec.cpp  
https://os.mbed.com/users/xshige/code/Sparkfun_CC3000_WiFi_OSCtranceiver//file/e62251d890c1/main.cpp/   

[Wio Lite RISC-V(GD32VF103)](http://akizukidenshi.com/catalog/g/gM-14785/)   

[Wio Lite RISC V GD32VF103 with ESP8266](https://wiki.seeedstudio.com/Wio_Lite_RISC_V_GD32VF103_with_ESP8266/)  
[Blink with a Wio Lite RISC-V with ESP8266](https://www.sigmdel.ca/michel/ha/gd32v/wio_lite_01_en.html)  

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

[ESP8266をTCPサーバとTCPクライアントにするATコマンド実例 (1/4)](https://monoist.atmarkit.co.jp/mn/articles/1510/19/news007.html)  
[esp8266_at_command_examples_en.pdf](https://www.espressif.com/sites/default/files/documentation/4b-esp8266_at_command_examples_en.pdf)  

[ESP8266をWifiモデムとして使う - ATコマンドによるMQTT通信](http://nopnop2002.webcrow.jp/ESP-AT-Firmware/ESP-AT-Firmware-31.html)  
[MQTT_via_ESP01](https://github.com/nopnop2002/MQTT_via_ESP01)   
[Arduino WiFi library for ESP8266 modules](https://github.com/bportaluri/WiFiEsp)  

[espressif/ESP8266_AT - examples](https://github.com/espressif/ESP8266_AT/wiki/at_example_0020000903)   
[Espressif Documentation](https://www.espressif.com/en/support/documents/technical-documents)  

以上
