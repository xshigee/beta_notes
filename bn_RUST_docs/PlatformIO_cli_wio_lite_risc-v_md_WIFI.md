
2020/6/20:  
初版

PlatformIO Wio Lite RISC-V WiFi
# PlatformIO Wio Lite RISC-V WiFi

## 概要
Wio_Lite_RISC-VボードでWiFiを動かす。
これは[「開発ツールPlatformIOをcli(comand line interface)で使う(Wio_Lite_RISC-V版)」](https://beta-notes.way-nifty.com/blog/2020/06/post-f72459.html)の続きになる。    
開発ツール(PlatformIOなど)のインストールについては[「開発ツールPlatformIOをcli(comand line interface)で使う(Wio_Lite_RISC-V版)」](https://beta-notes.way-nifty.com/blog/2020/06/post-f72459.html)を参照のこと、ここでは、WiFiを動かすためのプログラムについて記載する。
(ホストPCとしてはubuntuを想定している)


## telnet_test

src/telnet_test.cpp
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

void setup() 
{
  wio_serial_init();
  //Serial.begin(115200);  //serial port of GD32
  //Serial1.begin(115200);  //serial port of ESP8266
  pinMode(LED_BUILTIN, OUTPUT);
 
  delay(3000);

  usart0_print("\r\n\r\n\r\n\r\n\r\n");
 
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

void loop()
{
  digitalWrite(LED_BUILTIN, HIGH); // debug: for indicate in-loop

  // test loop
  while(true) {
    if ( usart_flag_get(USART1, USART_FLAG_RBNE)== SET) _put0_char(_get1_char());
    if ( usart_flag_get(USART0, USART_FLAG_RBNE)== SET) _put1_char(_get0_char());
 };

}
 ```
以下については、自分の環境に合わせて変更すること：  
#define MY_SSID "your_ssid"  
#define MY_PASSWD "your_passwd"   

ボードのPA9(TX). PA10(RX)、GNDをUSBシリアルに接続する。
書き込み後に「picocom /dev/ttyACM0 -b115200」で通信ソフトを起動すると以下のような出力が表示される：
```bash

$ picocom /dev/ttyACM0 -b115200


SEND:
AT+CWQAP
RES:
AT+CWQAP
I (2991) wifi: state: 5 -> 0 (0)

OK
I (3117) wifi: state: 0 -> 2 (b0)
I (3122) wifi: state: 2 -> 3 (0)
I (3127) wifi: state: 3 -> 5 (10)
WIFI CONNECTED
WIFI GOT IP

-----------
SEND:
AT+CWMODE=1
RES:
AT+CWMODE=1

OK

-----------
SEND:
AT+CWJAP="your_ssid","your_passwd"
RES:
AT+CWJAP="your_ssid","your_passwd"
I (13687) wifi: state: 5 -> 0 (0)
WIFI DISCONNECT
I (13860) wifi: state: 0 -> 2 (b0)
I (13866) wifi: state: 2 -> 3 (0)
I (13881) wifi: state: 3 -> 5 (10)
WIFI CONNECTED
WIFI GOT IP

OK

-----------
SEND:
AT+CIFSR
RES:
AT+CIFSR
+CIFSR:STAIP,"192.168.0.20"
+CIFSR:STAMAC,"2c:f4:32:48:ed:0b"

OK

-----------
SEND:
AT+CIPMUX=1
RES:
AT+CIPMUX=1

OK

-----------
SEND:
AT+CIPSERVER=1,80
RES:
AT+CIPSERVER=1,80

OK

-----------

ipAddress: 192.168.0.20
```
ここで、Wio-LiteのIPが「192.168.0.20」であることが分かる。（言うまでもないが、ネット環境によって、このIPは変わる）  
以下のようにtelnetでアクセスしてみる。
```

$ telnet 192.168.0.20 80
Trying 192.168.0.20...
Connected to 192.168.0.20.
Escape character is '^]'.
hello telnet access
```
これに対応してWio-Lite側には以下が出力される：
```

ipAddress: 192.168.0.20

0,CONNECT

+IPD,0,21:hell0 telnet access
```

また、以下のように「AT+CIPSEND」コマンドを使用して
Wio-Liteからtelnet側にテキスト「HELLO」を送ることができる。

Wio-Lite側：
```

AT+CIPSEND=0,7　#行末に[Enter],[Control-J]を入力する
#「0,7」は、リンク0に7文字送信することを意味する

>HELLO　#エコーされないので、実際には表示されないが、行末に[Enter],[Control-J]を入力する。
SEND OK
```

ちなみに、XIAOをUSBシリアルとして使用する場合は以下のように接続する：
| wio-lite-rv | XIAO |
| ---: | :--- |
| PA10(RX) | D6(TX) |
| PA9(TX) | D7(RX) |
| GND | GND |


## web_test
LEDをオン/オフするwebサーバープログラム：   
src/web_test.cpp
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
 
char html[50];
char command[20];
 
char LED[30];
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
  //for(int cc=0; cc<500000; cc++) {
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

void dispatch(void) {

  bool foundIPD = false;
  for (int i=0; i<(int)strlen(res); i++)
  {
    if (  (res[i]=='I') && (res[i+1]=='P') && (res[i+2]=='D')   ) { foundIPD = true;    }
  }
  
  if ( foundIPD  )  
  {
  
    bool LEDstate = false;
    int LEDstatepos = 0;
    for (int i=0; i<(int)strlen(res); i++)
    {
      if (!LEDstate) // just get the first occurrence of name
      {
        if ( (res[i]=='L') &&  (res[i+1]=='E')&&  (res[i+2]=='D')&&  (res[i+3]=='=')) 
        { 
          LEDstate = true;
          LEDstatepos = i+4;
        }
        else LEDstatepos = 0;      
      }     
    }


    if (LEDstate)
    {
      int tempPos = 0;
      bool finishedCopying = false;
      for (int i= LEDstatepos; i<(int)strlen(res); i++)
      {
        if ( (res[i]==' ') && !finishedCopying )  { finishedCopying = true;   } 
        if ( !finishedCopying )  { LED[tempPos] = res[i];   tempPos++; }
      }              
      LED[tempPos] = 0;
    }

    if (LEDstate) {
       usart0_print( "LED state = ");
       usart0_println(LED);
    } 
    else {
      usart0_println( "format incorrect");  
    } 
 
   // display LED status
   usart0_print("LED:");
   usart0_println(LED);

   if (strcmp(LED,"on")==0) {digitalWrite(LED_BUILTIN, HIGH); }
   else if (strcmp(LED ,"off")==0) {digitalWrite(LED_BUILTIN, LOW); }
   else if (strcmp(LED,"ON")==0) {digitalWrite(LED_BUILTIN, HIGH); }
   if (strcmp(LED ,"OFF")==0) {digitalWrite(LED_BUILTIN, LOW); }


   strcpy(html,"<html><head></head><body>");
   //strcpy(command,"AT+CIPSEND=0,25\r\n");
   sprintf(command,"AT+CIPSEND=0,%d\r\n",(int)strlen(html));
   //
   usart0_print("DEBUG-CMD:\r\n");
   usart0_print(command);
   usart0_print("\r\n-------------\r\n");
   usart0_print("DEBUG-HTML:\r\n");
   usart0_print(html);
   usart0_print("\r\n=============\r\n\r\n");
   //
   usart1_print(command);
   get_resCmd();

   usart1_print(html);
   get_resHtml();
 
   strcpy(html,"<h1>LED Test</h1>");
   //strcpy(command,"AT+CIPSEND=0,17\r\n");
   sprintf(command,"AT+CIPSEND=0,%d\r\n",(int)strlen(html));
   //
   usart0_print("DEBUG-CMD:\r\n");
   usart0_print(command);
   usart0_print("\r\n-------------\r\n");
   usart0_print("DEBUG-HTML:\r\n");
   usart0_print(html);
   usart0_print("\r\n=============\r\n\r\n");
   //
   usart1_print(command); 
   get_resCmd();
      
   usart1_print(html);
   get_resHtml();
 
   strcpy(html,"<p>LED Statment</p>");
   //strcpy(command,"AT+CIPSEND=0,19\r\n");
   sprintf(command,"AT+CIPSEND=0,%d\r\n",(int)strlen(html));
   //
   usart0_print("DEBUG-CMD:\r\n");
   usart0_print(command);
   usart0_print("\r\n-------------\r\n");
   usart0_print("DEBUG-HTML:\r\n");
   usart0_print(html);
   usart0_print("\r\n=============\r\n\r\n");
   //
   usart1_print(command);  
   get_resCmd();
     
   usart1_print(html);
   get_resHtml();
 
   if (LEDstate)
   {
     // write name
        strcpy(html,"<p>LED state is "); strcat(html, LED ); strcat(html,"</p>");

    // need the length of html for the cipsend command
/**
    lenHtml = strlen( html );
    strcpy(command,"AT+CIPSEND=0,"); __itoa( lenHtml, temp, 10);
    strcat(command, temp); strcat(command, "\r\n");
**/
    sprintf(command,"AT+CIPSEND=0,%d\r\n",(int)strlen(html));
    //
    usart0_print("DEBUG-CMD:\r\n");
    usart0_print(command);
    usart0_print("\r\n-------------\r\n");
    usart0_print("DEBUG-HTML:\r\n");
    usart0_print(html);
    usart0_print("\r\n=============\r\n\r\n");
    //
    usart1_print(command);
    get_resCmd();
          
    usart1_print(html);
    get_resHtml();                           
  }

/**
  strcpy(html,"<form action=\""); 
  strcat(html, ipAddress); 
  strcat(html, "\" method=\"GET\">"); strcat(command, "\r\n");
**/
  sprintf(html,"<form action=\"%s\" method=\"GET\">\r\n",ipAddress);

/** 
  lenHtml = strlen( html );
  strcpy(command,"AT+CIPSEND=0,"); 
  __itoa( lenHtml, temp, 10); 
  strcat(command, temp);  
  strcat(command, "\r\n");
**/
  sprintf(command,"AT+CIPSEND=0,%d\r\n",(int)strlen(html));
  //
  usart0_print("DEBUG-CMD:\r\n");
  usart0_print(command);
  usart0_print("\r\n-------------\r\n");
  usart0_print("DEBUG-HTML:\r\n");
  usart0_print(html);
  usart0_print("\r\n=============\r\n\r\n");
  //
  usart1_print(command);
  get_resCmd();
          
  usart1_print(html);
  get_resHtml();
 
  strcpy(html,"LEDstate:<br><input type=\"text\" name=\"LED\">");
  //strcpy(command,"AT+CIPSEND=0,43\r\n");
  sprintf(command,"AT+CIPSEND=0,%d\r\n",(int)strlen(html));
  //
  usart0_print("DEBUG-CMD:\r\n");
  usart0_print(command);
  usart0_print("\r\n-------------\r\n");
  usart0_print("DEBUG-HTML:\r\n");
  usart0_print(html);
  usart0_print("\r\n=============\r\n\r\n");
  //
  usart1_print(command);
  get_resCmd();

  usart1_print(html);   
  get_resHtml();         
 
  strcpy(html,"<input type=\"submit\" value=\"Submit\"></form>");
  //strcpy(command,"AT+CIPSEND=0,43\r\n");
  sprintf(command,"AT+CIPSEND=0,%d\r\n",(int)strlen(html));
  //
  usart0_print("DEBUG-CMD:\r\n");
  usart0_print(command);
  usart0_print("\r\n-------------\r\n");
  usart0_print("DEBUG-HTML:\r\n");
  usart0_print(html);
  usart0_print("\r\n=============\r\n\r\n");
  //
  usart1_print(command);
  get_resCmd(); 
      
  usart1_print(html);
  get_resHtml();   
 
  strcpy(html,"</body></html>");
  //strcpy(command,"AT+CIPSEND=0,14\r\n");
  sprintf(command,"AT+CIPSEND=0,%d\r\n",(int)strlen(html));
  //
  usart0_print("DEBUG-CMD:\r\n");
  usart0_print(command);
  usart0_print("\r\n-------------\r\n");
  usart0_print("DEBUG-HTML:\r\n");
  usart0_print(html);
  usart0_print("\r\n=============\r\n\r\n");
  //
  usart1_print(command);
  get_resCmd();
 
  usart1_print(html);
  get_resHtml(); 
 
  strcpy(command,"AT+CIPCLOSE=0\r\n");
  //
  usart0_print("DEBUG-CMD:\r\n");
  usart0_print(command);
  usart0_print("\r\n=============\r\n\r\n");
  //
  usart1_print(command);
  get_resCmd(); 
 
 } // if(espSerial.find("+IPD"))

 // drop to here and wait for next request.
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

ボードのPA9(TX). PA10(RX)、GNDをUSBシリアルに接続する。   
「picocom /dev/ttyACM0 -b115200」を起動して以下のコマンドを送る：  
(通信ソフトを起動した後、ボードを接続してプログラムを起動する)

```

$ picocom /dev/ttyACM0 -b115200


...
<省略>
...
OK

-----------

ipAddress: 192.168.0.6

# ここで、上で取得したIPと設定したportである80であるIP:port(192.168.0.6:80)にブラウザーでアクセスすると
# web画面が表示される

....................


RES:

OK

+IPD,1,406:GET /favicon.ico HTTP/1.1
Host: 192.168.0.6
Connection: keep-alive
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.80 Safari/537.36
Sec-Metadata: destination=image, site=same-origin
Accep
format incorrect
LED:
DEBUG-CMD:
AT+CIPSEND=0,25

-------------
DEBUG-HTML:
<html><head></head><body>
=============

DEBUG-CMD:
AT+CIPSEND=0,17

-------------
DEBUG-HTML:
<h1>LED Test</h1>
=============

DEBUG-CMD:
AT+CIPSEND=0,19

-------------
DEBUG-HTML:
<p>LED Statment</p>
=============

DEBUG-CMD:
AT+CIPSEND=0,40

-------------
DEBUG-HTML:
<form action="192.168.0.6" method="GET">
=============

DEBUG-CMD:
AT+CIPSEND=0,43

-------------
DEBUG-HTML:
LEDstate:<br><input type="text" name="LED">
=============

DEBUG-CMD:
AT+CIPSEND=0,43

-------------
DEBUG-HTML:
<input type="submit" value="Submit"></form>
=============

DEBUG-CMD:
AT+CIPSEND=0,14

-------------
DEBUG-HTML:
</body></html>
=============

DEBUG-CMD:
AT+CIPCLOSE=0

=============
```
web画面で入力欄にonまたはoffを入力して
[Submit]を押すと、on/offに応じて
オンボードのLEDが光る。   
または、以下のようなurlでアクセスしてもLEDをon/offできる：
```
http://192.168.0.6/192.168.0.6?LED=on
http://192.168.0.6/192.168.0.6?LED=off
```
IPが「192.168.0.6」の場合なので、実際に使うときは割り当てられたIPでアクセスすること。


## 参考情報

[Wio Lite RISC-V(GD32VF103)](http://akizukidenshi.com/catalog/g/gM-14785/)   

[Wio Lite RISC V GD32VF103 with ESP8266](https://wiki.seeedstudio.com/Wio_Lite_RISC_V_GD32VF103_with_ESP8266/)  
[Blink with a Wio Lite RISC-V with ESP8266](https://www.sigmdel.ca/michel/ha/gd32v/wio_lite_01_en.html)  

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

[ESP8266をTCPサーバとTCPクライアントにするATコマンド実例 (1/4)](https://monoist.atmarkit.co.jp/mn/articles/1510/19/news007.html)  
[esp8266_at_command_examples_en.pdf](https://www.espressif.com/sites/default/files/documentation/4b-esp8266_at_command_examples_en.pdf)  

以上
