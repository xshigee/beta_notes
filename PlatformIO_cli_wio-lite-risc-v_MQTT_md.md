
2020/6/28:  
初版

PlatformIO Wio Lite RISC-V WiFi4 MQTT
# PlatformIO Wio Lite RISC-V WiFi4 MQTT

## 概要
Wio_Lite_RISC-VボードでWiFiを動かす（その4：MQTT）。
これは[「Wio_Lite_RISC-VボードでWiFiを動かす（その3：OSC）」](https://beta-notes.way-nifty.com/blog/2020/06/post-639150.html)の続きになる。ここでは、MQTTのデモプログラムについて記載する    
開発ツール(PlatformIOなど)のインストールについては[「開発ツールPlatformIOをcliで使う(Wio_Lite_RISC-V版)」](https://beta-notes.way-nifty.com/blog/2020/06/post-f72459.html)を参照のこと、ここでは、WiFiを動かすためのプログラムについて記載する。
(ホストPCとしてはubuntuを想定している)
   
プログラムにある以下の関数に関しては、
[https://github.com/nopnop2002/MQTT_via_ESP01/blob/master/MQTT_Subscribe_ESP01/MQTT_Subscribe_ESP01.ino](https://github.com/nopnop2002/MQTT_via_ESP01/blob/master/MQTT_Subscribe_ESP01/MQTT_Subscribe_ESP01.ino)にあるものを、そのまま流用した：   
・int buildConnect(char *buf, int keep_alive, char *client_id, char *will_topic, char *will_msg)  
・int buildSubscribe(char *buf, char *topic)   
・int buildPublish(char *buf, char *topic, char *msg)   


## 接続
wio-lite-rvボードのPA9(TX). PA10(RX)、GNDをUSBシリアルに接続する。   
ちなみに、XIAOをUSBシリアルとして使用する場合は以下のように接続する：  
| wio-lite-rv | XIAO |
| ---: | :--- |
| PA10(RX) | D6(TX) |
| PA9(TX) | D7(RX) |
| GND | GND |

## MQTT_pub_test
MQTT送信プログラム：  
src/MQTT_pub_test.cpp
```c++


#include <Arduino.h>
#include <stdio.h>
#include <stdarg.h>
#include <math.h>

#define MY_SSID "your_ssid"
#define MY_PASSWD "your_passwd"

#define TARGET_IP "test.mosquitto.org"
#define OUT_PORT 1883
#define IN_PORT  1883

char ipAddress [20];

#define CLIENT_ID "fedx" // unique string(you can use MAC address) You can change
#define MQTT_KEEP_ALIVE 60
#define MQTT_WILL_TOPIC "ESP8266AT-MQTT/"       // You can change
#define MQTT_WILL_MSG   "I am leaving..."       // You can change

//----- MSTT con/sub/pub func -----
char con[20];
char sub[100];
char pub[100];
int msize, plen, slen;

int buildConnect(char *buf, int keep_alive, char *client_id, char *will_topic, char *will_msg) {
  int rlen = 12;
  int pos = 14;

  int client_id_len = strlen(client_id);
  //Serial.println(client_id_len);
  buf[pos++] = 0x00;
  buf[pos++] = client_id_len;
  for(int i=0;i<client_id_len;i++) {
    buf[pos++] = client_id[i];
  }
  rlen = rlen + 2 + client_id_len;
  
  int will_topic_len = strlen(will_topic);
//  Serial.print("will_topic_len=");
//  Serial.println(will_topic_len);
  int will_msg_len = strlen(will_msg);
//  Serial.print("will_msg_len=");
//  Serial.println(will_msg_len);

  if (will_topic_len > 0 && will_msg_len > 0) {
    buf[pos++] = 0x00;
    buf[pos++] = will_topic_len;
    for(int i=0;i<will_topic_len;i++) {
      buf[pos++] = will_topic[i];
    }
    buf[pos++] = 0x00;
    buf[pos++] = will_msg_len;
    for(int i=0;i<will_msg_len;i++) {
      buf[pos++] = will_msg[i];
    }
    rlen = rlen + 2 + will_topic_len + 2 + will_msg_len;  
  }

  buf[0] = 0x10;
  buf[1] = rlen;
  buf[2] = 0x00;
  buf[3] = 0x06;
  buf[4] = 'M';
  buf[5] = 'Q';
  buf[6] = 'I';
  buf[7] = 's';
  buf[8] = 'd';
  buf[9] = 'p';
  buf[10] = 0x03;
  buf[11] = 0x02;
  if (will_topic_len > 0 && will_msg_len > 0) buf[11] = 0x06;
  buf[12] = 0x00;
  buf[13] = keep_alive;
  return buf[1] + 2;  
}

int buildSubscribe(char *buf, char *topic) {
  int tlen = strlen(topic);
  for(int i=0;i<tlen;i++) {
    buf[6+i] = topic[i];
  }
  buf[0] = 0x82;
  buf[1] = tlen + 5;
  buf[2] = 0x00;
  buf[3] = 0x01;
  buf[4] = 0x00;
  buf[5] = tlen;
  buf[tlen+6] = 0x00;
  return buf[1] + 2;   
}

int buildPublish(char *buf, char *topic, char *msg) {
  int tlen = strlen(topic);
  for(int i=0;i<tlen;i++) {
    buf[4+i] = topic[i];
  }
  int mlen = strlen(msg);
  for(int i=0;i<mlen;i++) {
    buf[4+tlen+i] = msg[i];
  }
  buf[0] = 0x30;
  buf[1] = tlen + mlen + 2;
  buf[2] = 0x00;
  buf[3] = tlen;
  return buf[1] + 2;   
}

//------------------------------

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

//---------------------------------

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
  usart0_print("AT+CIPSERVER=1,80\r\n"); ;
  usart1_print("AT+CIPSERVER=1,80\r\n"); 
  get_res();
  usart0_println("RES:");
  usart0_println(res);
  usart0_println("-----------");

  usart0_printf("\r\nipAddress: %s\r\n\r\n",ipAddress); 

//------------------------------------------

  // TCP access
  usart0_printf("AT+CIPSTART=1,\"TCP\",\"%s\",%d\r\n",TARGET_IP,OUT_PORT);
  usart1_printf("AT+CIPSTART=1,\"TCP\",\"%s\",%d\r\n",TARGET_IP,OUT_PORT);
  get_res();
  usart0_println("RES:");
  usart0_println(res);
  usart0_println("-----------");

  // build connect info
  msize = buildConnect(con,MQTT_KEEP_ALIVE,CLIENT_ID,MQTT_WILL_TOPIC,MQTT_WILL_MSG);
  // send connect info
  usart0_printf("AT+CIPSEND=1,%d\r\n",msize);
  usart1_printf("AT+CIPSEND=1,%d\r\n",msize);
  get_res();
  usart0_println("RES:");
  usart0_println(res);
  usart0_println("-----------");
  // send connect info
  for (int i=0;i<msize;i++) _put1_char(con[i]);  
  for (int i=0;i<msize;i++) _put0_char(con[i]);
  _put0_char('\r'); _put0_char('\n');                   
  get_res();

}

//------------

void dispatch(void)
{
  // setup string pointer table
  char *h[20]; for(int i=0; i<20; i++) h[i]=NULL;
  h[0] = &res[0];
  int pi = 1;
  int cp;

  int len = (int)strlen(res); // keep length of res[]
  for (int i=0; i<len; i++) {
     //if ((res[i]=='I') && (res[i+1]=='P') && (res[i+2]=='D') ) { // check IPD
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
    if (h[i] != NULL) usart0_print(h[i]);
  }
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

  // publishing
 
 // message#0
  plen = buildPublish(pub, "topic0", "msg-hello00");
  usart0_printf("AT+CIPSEND=1,%d\r\n",plen);
  usart1_printf("AT+CIPSEND=1,%d\r\n",plen);
  get_res();
  // send pub info
  for (int i=0;i<plen;i++) _put1_char(pub[i]);                  
  get_res();
  usart0_println("pub-res:");
  usart0_println(res);
  usart0_println("-----------");

  // message#1
  plen = buildPublish(pub, "topic0", "msg-hello01");
  usart0_printf("AT+CIPSEND=1,%d\r\n",plen);
  usart1_printf("AT+CIPSEND=1,%d\r\n",plen);
  get_res();
  // send pub info
  for (int i=0;i<plen;i++) _put1_char(pub[i]);                  
  get_res();
  usart0_println("pub-res:");
  usart0_println(res);
  usart0_println("-----------");

  // message#2
  plen = buildPublish(pub, "topic0", "msg-hello02");
  usart0_printf("AT+CIPSEND=1,%d\r\n",plen);
  usart1_printf("AT+CIPSEND=1,%d\r\n",plen);
  get_res();
  // send pub info
  for (int i=0;i<plen;i++) _put1_char(pub[i]);                  
  get_res();
  usart0_println("pub-res:");
  usart0_println(res);
  usart0_println("-----------");

     }
  }

}

 ```
以下については、自分の環境に合わせて変更すること：  
```

#define MY_SSID "your_ssid"  
#define MY_PASSWD "your_passwd"  
```
以下は利用するMQTTサーバー(broker)に合わせる：
```
#define TARGET_IP "test.mosquitto.org"
#define OUT_PORT 1883
#define IN_PORT  1883 

```

書き込み後に「picocom /dev/ttyACM0 -b115200」で通信ソフトを起動すると以下のような出力が表示される：
```bash


$ picocom /dev/ttyACM0 -b115200


SEND:
AT+RST
RES:
AT+RST

OK
I (2989) wifi: state: 5 -> 0 (0)
WIFI DISCONNECT
sl
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

ipAddress: 192.168.0.18

AT+CIPSTART=1,"TCP","test.mosquitto.org",1883
RES:
AT+CIPSTART=1,"TCP","test.mosquitto.org",1883
1,CONNECT

OK

...
<省略>

```

## 対向する送受信プログラム
以下をブラウザ(chromeのみ)で動かす：

MQTT_AT_webTest.html
```javascript


<html>

<script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>
<body>

<script>

document.body.innerHTML = '';
var consout = 'MQTT over WebSockets Test'+'<br>'
document.body.innerHTML = consout;

//var mqtt = require('mqtt')
var client = mqtt.connect('ws://test.mosquitto.org:8081')

// subscribe Topic
client.subscribe('topic0');
//client.subscribe('topic0/sample/hello');
//client.subscribe('topic0/sample/#');
// wildcard topic
//client.subscribe('topic0/#');
//client.subscribe('#');

client.on('message', function(topic, payload) {
  console.log([topic, payload].join(': '));
  consout += [topic, payload].join(': ')+'<br>'
  document.body.innerHTML = consout
  // disconnect
  //client.end();
});

// publish messages
client.publish('topic0', 'hello world of MQTT! #1');
client.publish('topic0', 'hello world of MQTT! #2');
client.publish('topic0', 'hello world of MQTT! #3');
client.publish('topic0', 'hello world of MQTT! #4');
client.publish('topic0', 'hello world of MQTT! #5');

</script>

</body>

</html>
```

上をプラウザーで起動した後、
wio-liteの送信プログラムを動作させる。

以下のweb画面が表示される(例)：
```
MQTT over WebSockets Test
topic0: Bienvenue sur mosquitto.org
topic0: hello world of MQTT! #1　　　# webアプリで送信(publish)したものを受信している
topic0: hello world of MQTT! #2　　　# webアプリで送信(publish)したものを受信している
topic0: hello world of MQTT! #3　　　# webアプリで送信(publish)したものを受信している
topic0: hello world of MQTT! #4　　　# webアプリで送信(publish)したものを受信している
topic0: hello world of MQTT! #5　　　# webアプリで送信(publish)したものを受信している
topic0: msg-hello00　# wio-liteで送信(publish)したものを受信している
topic0: msg-hello01　# wio-liteで送信(publish)したものを受信している
topic0: msg-hello02　# wio-liteで送信(publish)したものを受信している
topic0: msg-hello00　# wio-liteで送信(publish)したものを受信している
topic0: msg-hello01　# wio-liteで送信(publish)したものを受信している
topic0: msg-hello02　# wio-liteで送信(publish)したものを受信している
topic0: msg-hello00　# wio-liteで送信(publish)したものを受信している
topic0: msg-hello01　# wio-liteで送信(publish)したものを受信している
topic0: msg-hello02　# wio-liteで送信(publish)したものを受信している
topic0: msg-hello00　# wio-liteで送信(publish)したものを受信している
topic0: msg-hello01　# wio-liteで送信(publish)したものを受信している
topic0: msg-hello02　# wio-liteで送信(publish)したものを受信している
...

```

## MQTT_sub_test
MQTT受信プログラム：  
src/MQTT_sub_test.cpp
```c++


#include <Arduino.h>
#include <stdio.h>
#include <stdarg.h>
#include <math.h>

#define MY_SSID "your_ssid"
#define MY_PASSWD "your_passswd"

#define TARGET_IP "test.mosquitto.org"
#define OUT_PORT 1883
#define IN_PORT  1883

char ipAddress [20];

#define CLIENT_ID "fedx" // unique string(you can use MAC address) You can change
#define MQTT_KEEP_ALIVE 3600 // 60
#define MQTT_WILL_TOPIC "ESP8266AT-MQTT/"       // You can change
#define MQTT_WILL_MSG   "I am leaving..."       // You can change

//----- MSTT con/sub/pub func -----
char con[20];
char sub[100];
char pub[100];
int msize, plen, slen;

int buildConnect(char *buf, int keep_alive, char *client_id, char *will_topic, char *will_msg) {
  int rlen = 12;
  int pos = 14;

  int client_id_len = strlen(client_id);
  //Serial.println(client_id_len);
  buf[pos++] = 0x00;
  buf[pos++] = client_id_len;
  for(int i=0;i<client_id_len;i++) {
    buf[pos++] = client_id[i];
  }
  rlen = rlen + 2 + client_id_len;
  
  int will_topic_len = strlen(will_topic);
//  Serial.print("will_topic_len=");
//  Serial.println(will_topic_len);
  int will_msg_len = strlen(will_msg);
//  Serial.print("will_msg_len=");
//  Serial.println(will_msg_len);

  if (will_topic_len > 0 && will_msg_len > 0) {
    buf[pos++] = 0x00;
    buf[pos++] = will_topic_len;
    for(int i=0;i<will_topic_len;i++) {
      buf[pos++] = will_topic[i];
    }
    buf[pos++] = 0x00;
    buf[pos++] = will_msg_len;
    for(int i=0;i<will_msg_len;i++) {
      buf[pos++] = will_msg[i];
    }
    rlen = rlen + 2 + will_topic_len + 2 + will_msg_len;  
  }

  buf[0] = 0x10;
  buf[1] = rlen;
  buf[2] = 0x00;
  buf[3] = 0x06;
  buf[4] = 'M';
  buf[5] = 'Q';
  buf[6] = 'I';
  buf[7] = 's';
  buf[8] = 'd';
  buf[9] = 'p';
  buf[10] = 0x03;
  buf[11] = 0x02;
  if (will_topic_len > 0 && will_msg_len > 0) buf[11] = 0x06;
/*
  buf[12] = 0x00;
  buf[13] = keep_alive;
*/
  buf[12] = 0xff & (keep_alive >> 8);
  buf[13] = 0xff & keep_alive;
  return buf[1] + 2;  
}

int buildSubscribe(char *buf, char *topic) {
  int tlen = strlen(topic);
  for(int i=0;i<tlen;i++) {
    buf[6+i] = topic[i];
  }
  buf[0] = 0x82;
  buf[1] = tlen + 5;
  buf[2] = 0x00;
  buf[3] = 0x01;
  buf[4] = 0x00;
  buf[5] = tlen;
  buf[tlen+6] = 0x00;
  return buf[1] + 2;   
}

int buildPublish(char *buf, char *topic, char *msg) {
  int tlen = strlen(topic);
  for(int i=0;i<tlen;i++) {
    buf[4+i] = topic[i];
  }
  int mlen = strlen(msg);
  for(int i=0;i<mlen;i++) {
    buf[4+tlen+i] = msg[i];
  }
  buf[0] = 0x30;
  buf[1] = tlen + mlen + 2;
  buf[2] = 0x00;
  buf[3] = tlen;
  return buf[1] + 2;   
}

//------------------------------

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
int reslen; // received char number for res[] 

void get_res(void) {
  int cpos = 0; 
  for(int cc=0; cc<24000000; cc++) {
     while ( usart_flag_get(USART1, USART_FLAG_RBNE)== SET) {
       //res[cpos] = (char) _get1_char(); cpos++;
       res[cpos] = (char) usart_data_receive(USART1); cpos++;
     };
  };
  res[cpos] = 0;
  reslen = cpos; // save # of received chars
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
  reslen = cpos; // save # of received chars
}

//---------------------------------

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
  usart0_print("AT+CIPSERVER=1,80\r\n"); ;
  usart1_print("AT+CIPSERVER=1,80\r\n"); 
  get_res();
  usart0_println("RES:");
  usart0_println(res);
  usart0_println("-----------");

  usart0_printf("\r\nipAddress: %s\r\n\r\n",ipAddress); 

//------------------------------------------

  // TCP access
  usart0_printf("AT+CIPSTART=1,\"TCP\",\"%s\",%d\r\n",TARGET_IP,OUT_PORT);
  usart1_printf("AT+CIPSTART=1,\"TCP\",\"%s\",%d\r\n",TARGET_IP,OUT_PORT);
  get_res();
  usart0_println("RES:");
  usart0_println(res);
  usart0_println("-----------");

  // build connect info
  msize = buildConnect(con,MQTT_KEEP_ALIVE,CLIENT_ID,MQTT_WILL_TOPIC,MQTT_WILL_MSG);
  // send connect info
  usart0_printf("AT+CIPSEND=1,%d\r\n",msize);
  usart1_printf("AT+CIPSEND=1,%d\r\n",msize);
  get_res();
  usart0_println("RES:");
  usart0_println(res);
  usart0_println("-----------");
  // send connect info
  for (int i=0;i<msize;i++) _put1_char(con[i]);  
  for (int i=0;i<msize;i++) _put0_char(con[i]);
  _put0_char('\r'); _put0_char('\n');                   
  get_res();

  // send sub info
  slen = buildSubscribe(sub, "topic0");
  usart0_printf("AT+CIPSEND=1,%d\r\n",slen);
  usart1_printf("AT+CIPSEND=1,%d\r\n",slen);
  get_res();
  usart0_println("RES:");
  usart0_println(res);
  usart0_println("-----------");
  // send connect info
  for (int i=0;i<slen;i++) _put1_char(sub[i]);                  
  get_res();
 
}

//------------

void dispatch(void)
{
  char ph0,ph1,ph2,ph3;
  char rtopic[20],rmsg[50];
  int pd, ps; // payload dest/src index
  int cp;

  //int len = (int)strlen(res); // keep length of res[]
  int len = reslen; // get # of received chars
  for (int i=0; i<len; i++) {
     if ((res[i]=='+') && (res[i+1]=='I') && (res[i+2]=='P') && (res[i+3]=='D') ) { // check +IPD
        for (cp = i+4; cp<len; cp++) {
            if (res[cp]==':') break;
        }
        ph0 = res[cp+1]; // ph0 (Payload Header 0)
        ph1 = res[cp+2]; // topic_length + msg_length +2
        ph2 = res[cp+3];
        ph3 = res[cp+4]; // topic_length
        //usart0_printf("ph0:0x%x ph2:0x%x\r\n",(int)ph0,(int)ph2); // for debug
        //usart0_printf("ph1:%d ph3:%d\r\n",(int)ph1,(int)ph3);   // for debug
        ps = cp+5;
        for(pd=0; pd<ph3; pd++) {rtopic[pd]=res[ps]; ps++; }; 
        rtopic[pd] = 0;
        for(pd=0; pd<(ph1-ph3-2); pd++) {rmsg[pd]=res[ps]; ps++; }; 
        rmsg[pd] = 0;
        // display received topic,message
        usart0_printf("received %s: %s\r\n",rtopic,rmsg);
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
     //if ((int)strlen(res) != 0) {
     if (reslen != 0) {
         //usart0_print("\r\nRES:\r\n");
         //usart0_println(res);
         dispatch();
     } else {
         //usart0_print("."); // indicating input wait


     }
  }

}
```

「MQTT_AT_webTest.html」をプラウザで起動した後、
wio-liteの受信プログラムを動作させる。

書き込み後に「picocom /dev/ttyACM0 -b115200」で通信ソフトを起動すると以下のような出力が表示される：
```

$ picocom /dev/ttyACM0 -b115200


SEND:
AT+RST
RES:
AT+RST

OK
I (2989) wifi: state: 5 -> 0 (0)
WIFI DISCONNECT
sl
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

OK

#以下、ブラウザーをリフレッシュ(リロード)するたびにpublishされ
#受信したものが以下のように表示される：
>
-----------
received topic0: hello world of MQTT! #1
received topic0: hello world of MQTT! #2
received topic0: hello world of MQTT! #3
received topic0: hello world of MQTT! #4
received topic0: hello world of MQTT! #5
received topic0: hello world of MQTT! #1
received topic0: hello world of MQTT! #2
received topic0: hello world of MQTT! #3
received topic0: hello world of MQTT! #4
received topic0: hello world of MQTT! #5
received topic0: hello world of MQTT! #1
received topic0: hello world of MQTT! #2
received topic0: hello world of MQTT! #3
received topic0: hello world of MQTT! #4
received topic0: hello world of MQTT! #5
received topic0: hello world of MQTT! #1
received topic0: hello world of MQTT! #2
received topic0: hello world of MQTT! #3
received topic0: hello world of MQTT! #4
received topic0: hello world of MQTT! #5
received topic0: hello world of MQTT! #1
received topic0: hello world of MQTT! #2
received topic0: hello world of MQTT! #3
received topic0: hello world of MQTT! #4
received topic0: hello world of MQTT! #5
...
...

```

## 参考情報

MQTT test server @mosquitto.org
```
https://test.mosquitto.org/
The server listens on the following ports:
1883 : MQTT, unencrypted
8883 : MQTT, encrypted
8884 : MQTT, encrypted, client certificate required
8080 : MQTT over WebSockets, unencrypted
8081 : MQTT over WebSockets, encrypted
websocketで使用するurlは以下になる：
'ws://test.mosquitto.org:8081'
```

MQTT test server @eclipse.org
```
http://mqtt.eclipse.org/
This is a public test MQTT broker service.
It currently listens on the following ports:
1883 : MQTT over unencrypted TCP
8883 : MQTT over encrypted TCP
80 : MQTT over unencrypted WebSockets (note: URL must be /mqtt )
443 : MQTT over encrypted WebSockets (note: URL must be /mqtt )
websocketで使用するurlは以下になる：
'ws://mqtt.eclipse.org/mqtt:443'
```

[10 Free Public & Private MQTT Brokers(For Testing & Production)](https://mntolia.com/10-free-public-private-mqtt-brokers-for-testing-prototyping/)

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
