
2020/6/22+:  
初版

PlatformIO Wio Lite RISC-V WiFi2
# PlatformIO Wio Lite RISC-V WiFi2

## 概要
Wio_Lite_RISC-VボードでWiFiを動かす（その２：STARWARS,REST-API）。
これは[「Wio_Lite_RISC-VボードでWiFiを動かす」](https://beta-notes.way-nifty.com/blog/2020/06/post-e3a206.html)の続きになる。    
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


## starwars_test
telnetでASCIIアートの「StarWars」アニメを表示するプログラム：  
src/starwars_test.cpp
```c++


#include <Arduino.h>
#include <stdio.h>
#include <stdarg.h>

#define MY_SSID "your_ssid"
#define MY_PASSWD "your_passwd"

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

//char res[500];
char res[3000]; // need big buffer for StarWars
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

  // telnet access
  usart0_print("AT+CIPSTART=1,\"TCP\",\"towel.blinkenlights.nl\",23\r\n");
  usart1_print("AT+CIPSTART=1,\"TCP\",\"towel.blinkenlights.nl\",23\r\n");
/***
  get_res();
  usart0_println("RES:");
  usart0_println(res);
  usart0_println("-----------");
***/

/****
  usart0_print("AT+CIPSTART=\"TCP\",\"imaoca.webcrow.jp\",80\r\n");
  usart1_print("AT+CIPSTART=\"TCP\",\"imaoca.webcrow.jp\",80\r\n");
****/

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
     if ((res[i]=='+') && (res[i+1]=='I') && (res[i+2]=='P') && (res[i+3]=='D') ) { // check +IPD
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

ipAddress: 192.168.0.14

# StarWarsのASCIIアートのアニメが表示される

```
実行するとStarWarsのASCIIアートのアニメが表示される。


## restapi_test
RESI-APIでボードを制御するサーバープログラム。   
src/restapi_test.cpp
```c++


#include <Arduino.h>
#include <stdio.h>
#include <string.h> 
#include <stdarg.h>

#define MY_SSID "your_ssid"
#define MY_PASSWD "your_passwd"

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
 
char html[50];
char command[20];
 
//char LED[30];
int lenHtml = 0;
char temp[5];

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
void get_resCmd(void) {
  int cpos = 0; 
  for(int cc=0; cc<100000; cc++) {
     while ( usart_flag_get(USART1, USART_FLAG_RBNE)== SET) {
       //res[cpos] = (char) _get1_char(); cpos++;
       res[cpos] = (char) usart_data_receive(USART1); cpos++;
     };
  };
  res[cpos] = 0;
}
void get_resHtml(void) {
  int cpos = 0;
  for(int cc=0; cc<1000000; cc++) {
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

}
 
//------------

int count = 0;  // test variable

void dispatch(void) {

  bool foundIPD = false;
  for (int i=0; i<(int)strlen(res); i++)
  {
    if (  (res[i]=='I') && (res[i+1]=='P') && (res[i+2]=='D')) { foundIPD = true;}
  }

  bool foundGET = false;
  int pGET;
  if (foundIPD) {
    for (int i=0; i<(int)strlen(res); i++) {
      // check 'GET'
      if ((res[i]=='G') && (res[i+1]=='E') && (res[i+2]=='T')) {
         foundGET = true;
         pGET = i+3;
         break;
      }
    }
  }

  bool foundApi = false;
  char api[24];
  for(int i=0; i<24; i++) api[i]=0;
  int dp = 0;
  if (foundGET) {
    for (int i=pGET; i<(int)strlen(res); i++) {
      // check '/api/v1/'
       if ((res[i]=='/') && (res[i+1]=='a') && (res[i+2]=='p') && (res[i+3]=='i') &&
          (res[i+4]=='/') && (res[i+5]=='v') && (res[i+6]=='1') && (res[i+7]=='/') ) {
               foundApi = true;
               int sp = i+8;
               while (res[sp]!=' ') {
                  api[dp]= res[sp];
                  dp++; sp++;
               }
               api[dp]=0;
      } // if check '/api/v1/'
    }
  }

  if (foundApi) {
    usart0_printf("API: %s\r\n", api);
    //
    if(strcmp(api,"led=0")==0){ 
        usart0_print("LED:off\r\n");
        digitalWrite(LED_BUILTIN, LOW); 
        //
        sprintf(html,"{ \"LED\" : \"off\" }\r\n");
        sprintf(command,"AT+CIPSEND=0,%d\r\n",(int)strlen(html));
        usart0_print("DEBUG-CMD:\r\n");
        usart0_print(command);
        usart0_print("\r\n-------------\r\n");
        usart0_print("DEBUG-HTML:\r\n");
        usart0_print(html);
        usart0_print("\r\n=============\r\n\r\n");
        usart1_print(command);
        get_resCmd();
        usart1_print(html);
        get_resHtml();
    }
    if(strcmp(api,"led=1")==0){
        usart0_print("LED:on\r\n"); 
        digitalWrite(LED_BUILTIN, HIGH); 
        //
        sprintf(html,"{ \"LED\" : \"on\"  }\r\n");
        sprintf(command,"AT+CIPSEND=0,%d\r\n",(int)strlen(html));
        usart0_print("DEBUG-CMD:\r\n");
        usart0_print(command);
        usart0_print("\r\n-------------\r\n");
        usart0_print("DEBUG-HTML:\r\n");
        usart0_print(html);
        usart0_print("\r\n=============\r\n\r\n");
        usart1_print(command);
        get_resCmd();
        usart1_print(html);
        get_resHtml();
    }
    if(strcmp(api,"count")==0){
        usart0_printf("count:%d\r\n",count);
        //
        sprintf(html,"{ \"count\" : %d}\r\n", count);
        sprintf(command,"AT+CIPSEND=0,%d\r\n",(int)strlen(html));
        usart0_print("DEBUG-CMD:\r\n");
        usart0_print(command);
        usart0_print("\r\n-------------\r\n");
        usart0_print("DEBUG-HTML:\r\n");
        usart0_print(html);
        usart0_print("\r\n=============\r\n\r\n");
        usart1_print(command);
        get_resCmd();
        usart1_print(html);
        get_resHtml();
   }

  } // if (foundApi)

  // close connection (is needed for displaying JSON data on the web)
  strcpy(command,"AT+CIPCLOSE=0\r\n");
  //
  usart0_print("DEBUG-CMD:\r\n");
  usart0_print(command);
  usart0_print("\r\n=============\r\n\r\n");
  //
  usart1_print(command);
  get_resCmd(); 
  //-----

  count++; // update test variable

  return;
}

void loop()
{

/***
  // test loop
  while(true) {
    if ( usart_flag_get(USART1, USART_FLAG_RBNE)== SET) _put0_char(_get1_char());
    if ( usart_flag_get(USART0, USART_FLAG_RBNE)== SET) _put1_char(_get0_char());
 };
****/

  while(true) {
     get_resHtml();
     if ((int)strlen(res) != 0) {
         usart0_print("\r\nRES:\r\n");
         usart0_println(res);
         dispatch();
     } else {
         usart0_print("."); // indicating input wait
     }
  }

}
```
以下については、自分の環境に合わせて変更すること：  
#define MY_SSID "your_ssid"  
#define MY_PASSWD "your_passwd"  

「picocom /dev/ttyACM0 -b115200」を起動すると以下が表示される：  

```

$ picocom /dev/ttyACM0 -b115200


SEND:
AT+GMR
RES:
lAT+GMR
AT version:2.1.0.0-dev(d25f6d2 - Oct 15 2019 12:03:04)
SDK version:v3.2-192-g81655f39
compile time(525fbfe):Oct 17 2019 05:39:13
Bin version:2.0.0(WROOM-02)

...
<省略>
...

-----------

ipAddress: 192.168.0.14

.......................................................................
RES:
0,CONNECT

+IPD,0,88:GET /api/v1/led=1 HTTP/1.1
Host: 192.168.0.14
User-Agent: curl/7.60.0
Accept: */*


API: led=1
LED:on
DEBUG-CMD:
AT+CIPSEND=0,19

-------------
DEBUG-HTML:
{ "LED" : "on"  }

=============

DEBUG-CMD:
AT+CIPCLOSE=0

=============

...

...

```

起動してIPが確定した後、REST-APIをサポートしているので以下のようなコマンドでボードを制御できる：

```

$ curl 'http://192.168.0.14:80/api/v1/led=1'
{ "LED" : "on"  }
#ここの時点でオンボードのLEDが光る
$ curl 'http://192.168.0.14:80/api/v1/led=0'
{ "LED" : "off" }
#ここの時点でオンボードのLEDが消える
$ curl 'http://192.168.0.14:80/api/v1/led=1'
{ "LED" : "on"  }
#ここの時点でオンボードのLEDが光る
$ curl 'http://192.168.0.14:80/api/v1/count'
{ "count" : 18}
#現在の変数countの内容が表示される
$ curl 'http://192.168.0.14:80/api/v1/count'
{ "count" : 20}
#現在の変数countの内容が表示される
$ curl 'http://192.168.0.14:80/api/v1/count'
{ "count" : 22}
#現在の変数countの内容が表示される
$ curl 'http://192.168.0.14:80/api/v1/count'
{ "count" : 24}
#現在の変数countの内容が表示される

```
上で使用していると同じurlをウェブブラウザーに入力してアクセスしても
同様に制御できる。
IPが「192.168.0.14」の場合なので、実際に使うときは割り当てられたIPでアクセスすること。


## 参考情報

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
