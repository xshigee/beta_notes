
2021/3/10  
初版

platformio naitive build
# platformio naitive build

## 概要
platfomioを使ってnaitive(linux/windows)のプログラムをビルドする方法について述べる。   
ここでは、platformioがインストール済みを前提とする。  
実行環境はubuntuとwindows10を前提にする。  

## windows10の場合
ここでは既にscoopなどでplatformioがインストール済みとする。  
以下の２つのファイルをプロジェクト・デレクトリに用意する：  
platformio.ini
```

; PlatformIO Project Configuration File
;
;   Build options: build flags, source filter, extra scripting
;   Upload options: custom port, speed and extra flags
;   Library options: dependencies, extra library storages
;
; Please visit documentation for the other options and examples
; http://docs.platformio.org/page/projectconf.html
[env:windows_x86]
# Stable version
#platform = windows_x86
# Development version
platform = https://github.com/platformio/platform-windows_x86.git
```

src/main.c
```c

#include <stdio.h>

int main()
{
    printf("Hello World from PlatformIO!\n");
    return 0;
}
```

ビルド/実行の例：  
PowerShell
```

mkdir win
cd win
notepad platformio.ini
#上の内容になるように編集する。

mkdir src
notepad src/main.c
#上の内容になるように編集する。

pio run -t clean
#必要なツールのダウンロードとインストールが行われる。

# ビルド
pio run

# 実行
.\.pio\build\windows_x86\program.exe
#(実行・出力例)
Hello World from PlatformIO!

```

## ubuntuの場合
ここでは既にplatformioがインストール済みとする。  
以下の２つのファイルをプロジェクト・デレクトリに用意する：   
platformio.ini
```

; PlatformIO Project Configuration File
;
;   Build options: build flags, source filter, extra scripting
;   Upload options: custom port, speed and extra flags
;   Library options: dependencies, extra library storages
;
; Please visit documentation for the other options and examples
; http://docs.platformio.org/page/projectconf.html

[env:linux_x86_64]
# Stable version
#platform = linux_x86_64
# Development version
platform = https://github.com/platformio/platform-linux_x86_64.git

```

src/main.c
```c

#include <stdio.h>

int main()
{
    printf("Hello World from PlatformIO!\n");
    return 0;
}
```

ビルド/実行の例：   
bash
```

mkdir pc
cd pc
gedit platformio.ini
#上の内容になるように編集する。

mkdir src
gedit src/main.c
#上の内容になるように編集する。

pio run -t clean
#必要なツールのダウンロードとインストールが行われる。

# ビルド
pio run

# 実行
./.pio/build/linux_x86_64/program 
#(実行・出力例)
Hello World from PlatformIO!

```
## 参考情報

platformio関連：  
・[windows10にplatformioをインストールする(scoop版)](https://beta-notes.way-nifty.com/blog/2021/02/post-173bac.html)  
・[arduinoフレームワーク用platformio.ini集(linux版PlatformIOのインストール方法)](https://beta-notes.way-nifty.com/blog/2021/02/post-2b331d.html)  

・[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  
・[Arduino-IDEとPlatformioのコンパイラーの挙動の違いについて](https://beta-notes.way-nifty.com/blog/2020/07/post-fbe8f7.html)   
・[ubuntu20.04をインストールする](https://beta-notes.way-nifty.com/blog/2020/10/post-fa7773.html)   

以上
