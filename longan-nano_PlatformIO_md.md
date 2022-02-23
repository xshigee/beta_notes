
2019/12/27

Longan-Nano tool (VScode+PlatformIO)
# Longan-Nano Tool (VScode+PlatformIO)

## 概要
Longan-Nanoボードの開発ツールとして、VScodeの拡張としてPlatformIOを導入して、そのなかで開発ツール(コンパイラ、リンカ、アップローダー)をインストールする。
ここでは、linux環境でのインストール方法について説明する。サンプルとしては定番のblinkを動かしてみる。

## 準備
ツールをインストール前に環境整備として以下を設定する：  
(1)Python 2.7またはPython 3.5+のインストール
```bash
sudo apt-get install python3
```
(2)udevのruleの設定
```bash
curl -fsSL https://raw.githubusercontent.com/platformio/platformio-core/master/scripts/99-platformio-udev.rules | sudo tee /etc/udev/rules.d/99-platformio-udev.rules
sudo service udev restart
sudo usermod -a -G dialout $USER
sudo usermod -a -G plugdev $USER
```
(3)DFU(Device Firmuware Update)のインストール
```bash
sudo apt-get install dfu-util
```

## 開発ツールのインストール
上の準備が終わったら、プロセッサとボードの種類は異なるが、インストール方法が分かりやすいので、以下を参考にインストールする。：      
https://www.media.lab.uec.ac.jp/?page_id=1414  
ESP32をVSCodeとPlatformIO IDEで動かす方法   
   
今回の場合は以下の２つのうち１つを設定する：   
(1)frameworkを「arduino」にする場合   
platform = gd32v   
framework = arduino  
board = sipeed-longan-nano  

(2)frameworkを「sdk」にする場合   
platform = gd32v   
framework = gd32vf103-sdk   
board = sipeed-longan-nano  
  
## IDE設定
platformio.iniファイルを以下のように編集する(dfu関係が追加されただけで、ほとんどデフォルトと変わらない)：    
(1)frameworkを「arduino」にする場合
```
; PlatformIO Project Configuration File
;
;   Build options: build flags, source filter, extra scripting
;   Upload options: custom port, speed and extra flags
;   Library options: dependencies, extra library storages
;
; Please visit documentation for the other options and examples
; http://docs.platformio.org/page/projectconf.html

[env:sipeed-longan-nano]
platform = gd32v
framework = arduino
board = sipeed-longan-nano
monitor_speed = 115200
upload_protocol = dfu
```

(2)frameworkを「sdk」にする場合
```
; PlatformIO Project Configuration File
;
;   Build options: build flags, source filter, extra scripting
;   Upload options: custom port, speed and extra flags
;   Library options: dependencies, extra library storages
;
; Please visit documentation for the other options and examples
; http://docs.platformio.org/page/projectconf.html

[env:sipeed-longan-nano]
platform = gd32v
framework = gd32vf103-sdk
board = sipeed-longan-nano
monitor_speed = 115200
upload_protocol = dfu
```

## ビルドテストのためにインポートするプログラム
(1)frameworkが「arduino」の場合   
プロジェクト名(フォルダ名)：arduino-blink   
Blink.cpp
```cpp
/*
 * Blink
 * Turns on an LED on for one second,
 * then off for one second, repeatedly.
 */

#include <Arduino.h>

// Set LED_BUILTIN if it is not defined by Arduino framework
// #define LED_BUILTIN 2

void setup()
{
  // initialize LED digital pin as an output.
  pinMode(LED_BUILTIN, OUTPUT);
}

void loop()
{
  // turn the LED on (HIGH is the voltage level)
  digitalWrite(LED_BUILTIN, HIGH);
  // wait for a second
  delay(1000);
  // turn the LED off by making the voltage LOW
  digitalWrite(LED_BUILTIN, LOW);
  // wait for a second
  delay(1000);
}
```

(2)frameworkが「sdk」の場合   
プロジェクト名(フォルダ名)：longan-nano-blink   
main.c
```c
/*!
    \file  main.c
    \brief running led
    
    \version 2019-6-5, V1.0.0, firmware for GD32VF103
*/

/*
    Copyright (c) 2019, GigaDevice Semiconductor Inc.

    Redistribution and use in source and binary forms, with or without modification, 
are permitted provided that the following conditions are met:

    1. Redistributions of source code must retain the above copyright notice, this 
       list of conditions and the following disclaimer.
    2. Redistributions in binary form must reproduce the above copyright notice, 
       this list of conditions and the following disclaimer in the documentation 
       and/or other materials provided with the distribution.
    3. Neither the name of the copyright holder nor the names of its contributors 
       may be used to endorse or promote products derived from this software without 
       specific prior written permission.

    THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" 
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED 
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. 
IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, 
INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT 
NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR 
PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, 
WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) 
ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY 
OF SUCH DAMAGE.
*/

#include "gd32vf103.h"
#include "systick.h"
#include <stdio.h>

/* BUILTIN LED OF LONGAN BOARDS IS PIN PC13 */
#define LED_PIN GPIO_PIN_13
#define LED_GPIO_PORT GPIOC
#define LED_GPIO_CLK RCU_GPIOC

void longan_led_init()
{
    /* enable the led clock */
    rcu_periph_clock_enable(LED_GPIO_CLK);
    /* configure led GPIO port */ 
    gpio_init(LED_GPIO_PORT, GPIO_MODE_OUT_PP, GPIO_OSPEED_50MHZ, LED_PIN);

    GPIO_BC(LED_GPIO_PORT) = LED_PIN;
}

void longan_led_on()
{
    GPIO_BOP(LED_GPIO_PORT) = LED_PIN;
}

void longan_led_off()
{
    GPIO_BC(LED_GPIO_PORT) = LED_PIN;
}
/*!
    \brief      main function
    \param[in]  none
    \param[out] none
    \retval     none
*/
int main(void)
{
    longan_led_init();

    while(1){
        /* turn on builtin led */
        longan_led_on();
        delay_1ms(1000);
        /* turn off uiltin led */
        longan_led_off();
        delay_1ms(1000);
    }
}
```

## build/upload
(1)VScode画面の最下行にあるアイコンからBuidのアイコンをクリックしてビルド実行する。   
(2)ボードのホストＰＣに接続し、[boot]ボタンを押しながら[reset]ボタンを押して離す。これでDFUモードに入る。このあと、[boot]を離す。    
(3)VScode画面の最下行にあるアイコンからUploadのアイコンをクリックしてアップロード(ボードへのプログラム書込)を実行する。    
以下のようなログが出て、[FAILED]になるが、「File downloaded successfully」になっており、書き込みは正常に終了している。 
プログラムが正常に動作していることは、赤いLEDの点滅でも確認できる。(ソースを変更して周期を変えて動作させてもよい)

```bash
Opening DFU capable USB device...
ID 28e9:0189
Run-time device DFU version 011a
Claiming USB DFU Interface...
Setting Alternate Setting #0 ...
Determining device status: state = dfuERROR, status = 10
dfuERROR, clearing status
Determining device status: state = dfuIDLE, status = 0
dfuIDLE, continuing
DFU mode device DFU version 011a
Device returned transfer size 2048
GD32 flash memory access detected
Device model: GD32VF103CB
Memory segment (0x08000000 - 0801ffff)(rew)
Erase size 1024, page count 128
Downloading to address = 0x08000000, size = 6584
Download        [=========================] 100%         6584 bytes
Download done.
File downloaded successfully
dfu-util: dfuse_download: libusb_control_transfer returned -4
*** [upload] Error 74
==== [FAILED] Took 2.28 seconds =====
The terminal process terminated with exit code: 1
```

## フォルダ構成
デフォルトの設定では、home/Documents/PlatformIO/Projectsのなかにプロジェクト名のフォルダが生成される。

## 参考URL  
[Ｓｉｐｅｅｄ　Ｌｏｎｇａｎ　Ｎａｎｏ　ＲＩＳＣ－Ｖ　ＧＤ３２ＶＦ１０３ＣＢＴ６開発ボード](
http://akizukidenshi.com/catalog/g/gK-14678/)    
[A new generation ecosystem for embedded development](https://platformio.org/)   
[ESP32をVSCodeとPlatformIO IDEで動かす方法](https://www.media.lab.uec.ac.jp/?page_id=1414)


以上
