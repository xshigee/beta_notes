
2020/4/10

mbed-cli Install KL25Z
# mbed-cli Install KL25Z

## 概要
開発ツールmbed-cliをインストールする。ターゲットボードは以下のKL25Zとする。python2.7がツールのインストールに必要なので
そのための仮想環境を設定する。(pythonは、anacondaでインストールされている前提)  
(ホストPCとしてはubuntuを想定している)

[FRDM-KL25Z](https://os.mbed.com/platforms/KL25Z/)     

## 開発ツールのインストール
```bash

conda create -n mbed python=2.7
conda activate mbed
pip install mbed-cli

```
設定した仮想環境を確認するには以下を実行する：  
conda info -e  

この出力に「mbed」が含まれていればＯＫ。

インストール後も、本ツールを使用する場合
以下を実行する：  
conda activate mbed  



## デモ用プロジェクトhelloworldを作成する
```bash

mkdir helloworld
cd helloworld
mbed new .

```

## main.cppを作成する
```bash

nano main.cpp
```
以下の内容のmain.cppを作成する：  
main.cpp
```c++

// Colorful LED

#include <mbed.h>

#define DELAY 500

DigitalOut ledR(LED_RED);
DigitalOut ledG(LED_GREEN);
DigitalOut ledB(LED_BLUE);

int main(){
    while(true){
       // Orange
       ledR = 0;
       ledG = 0;
       ledB = -1;  
       Thread::wait(DELAY);
       // Megenta
       ledR = 0;
       ledG = -1;;
       ledB = 0;  
       Thread::wait(DELAY);
       // Red
       ledR = 0;
       ledG = -1;
       ledB = -1;  
       Thread::wait(DELAY);
       // Cyan
       ledR = -1;
       ledG = 0;
       ledB = 0;  
       Thread::wait(DELAY);
       // Green
       ledR = -1;
       ledG = 0;
       ledB = -1;  
       Thread::wait(DELAY);
       // Blue
       ledR = -1;
       ledG = -1;
       ledB = 0;  
       Thread::wait(DELAY);
       // Black
       ledR = -1;
       ledG = -1;
       ledB = -1;  
       Thread::wait(DELAY);
       // White
       ledR = 0;
       ledG = 0;
       ledB = 0;  
       Thread::wait(DELAY);

    }
}
```

## ビルド
```bash

mbed compile -v -t GCC_ARM -m kl25z

```

ビルド出力例：
```bash

...
...
Link: helloWorld
Elf2Bin: helloWorld
| Module           |         .text |       .data |        .bss |
|------------------|---------------|-------------|-------------|
| [fill]           |       44(+44) |       4(+4) |     28(+28) |
| [lib]/c.a        | 19072(+19072) | 2472(+2472) |     89(+89) |
| [lib]/gcc.a      |   8900(+8900) |       0(+0) |       0(+0) |
| [lib]/misc       |     192(+192) |       4(+4) |     28(+28) |
| main.o           |       80(+80) |       0(+0) |       0(+0) |
| mbed-os/drivers  |       92(+92) |       0(+0) |       0(+0) |
| mbed-os/hal      |   1436(+1436) |       4(+4) |     67(+67) |
| mbed-os/platform |   3044(+3044) |   264(+264) |   220(+220) |
| mbed-os/rtos     |   6460(+6460) |   168(+168) | 5973(+5973) |
| mbed-os/targets  |   2424(+2424) |       4(+4) |     19(+19) |
| Subtotals        | 41744(+41744) | 2920(+2920) | 6424(+6424) |
Total Static RAM memory (data + bss): 9344(+9344) bytes
Total Flash memory (text + data): 44664(+44664) bytes

Image: ./BUILD/KL25Z/GCC_ARM/helloWorld.bin
```

## 書き込み
上で作成したhelloworld.binをmbedストレージにドラッグ&ドロップする。

## 実行
書き込みが終わったら、ボードのresetボタンを押す。
ボードのLEDが色々な色で光れば動作としてはＯＫ．

## mbed-cli　コマンドリスト
```bash

$ mbed -h
usage: mbed [-h] [--version]             ...

Command-line code management tool for ARM mbed OS - http://www.mbed.com
version 1.10.2

Use "mbed <command> -h|--help" for detailed help.
Online manual and guide available at https://github.com/ARMmbed/mbed-cli

optional arguments:
  -h, --help         show this help message and exit
  --version          print version number and exit

Commands:
             
    new              Create new mbed program or library
    import           Import program from URL
    add              Add library from URL
    remove           Remove library
    deploy           Find and add missing libraries
    publish          Publish program or library
    update           Update to branch, tag, revision or latest
    sync             Synchronize library references
                     
    ls               View dependency tree
    releases         Show release tags
    status           Show version control status
                     
    compile          Compile code using the mbed build tools
    test             Find, build and run tests
    device-management
                     device management subcommand
    export           Generate an IDE project
    detect           Detect connected Mbed targets/boards
                     
    sterm            Open serial terminal to connected target.
                     
    config           Tool configuration
    target           Set or get default target
    toolchain        Set or get default toolchain
    cache            Repository cache management
                     
    help             This help screen
```
サポートしているボードを確認したい場合、変則的なやり方になるが
以下を実行するとサポートしているボードがリストアップされる:  
(意図的にエラーを出力させる)  
mbed compile -v -t GCC_ARM -m x   


## 参考情報

[mbedのプロジェクトをmbed-cliを使ってGCC-ARMでビルドするまで](https://tsubakicraft.wordpress.com/2017/07/06/mbed%E3%81%AE%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%82%92mbed-cli%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6gcc-arm%E3%81%A7%E3%83%93%E3%83%AB%E3%83%89%E3%81%99%E3%82%8B%E3%81%BE%E3%81%A7/)  
[Working with Mbed CLI](https://os.mbed.com/docs/mbed-os/v5.15/tools/working-with-mbed-cli.html)   

[主なMbed対応デバイスの紹介](https://www.aps-web.jp/academy/mbed/573/)  

[mbed FRDM KL25Z Examples](https://os.mbed.com/handbook/mbed-FRDM-KL25Z-Examples)  


[An introduction to Arm Mbed OS 5](https://os.mbed.com/docs/mbed-os/v5.15/introduction/index.html#tools)   
[Arm Mbed OS 2 vs. Arm Mbed OS 5](https://os.mbed.com/docs/mbed-os/v5.15/introduction/versions-and-releases.html)  


以上

