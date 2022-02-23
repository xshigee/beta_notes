
2021/3/1　　

platformio windows10
# platformio windows10

## 概要
windows10にplatformioをインストールする(scoop版)   
windows10にscoopでplatformioをインストールする方法について述べる。

## platformioをインストールする
PowerShell
```

# scoopをインストール
Set-ExecutionPolicy RemoteSigned -scope CurrentUser
iwr -useb get.scoop.sh | iex

# pythonをインストールする
scoop install python

pip3 install platformio

# (必要があれば)windows用VScodeをインストールする
scoop bucket add extras
scoop install vscode

# 別の機能が割り当てられているaliasを削除する
del alias:curl
del alias:wget

# curl, wgetをインストールする
scoop install curl
scoop install wget

# その他の必要となるツールをインストールする
scoop install git
```

scoopの簡単な説明:
```

# xxxxをインストール
scoop install xxxx

# Scoop自身とローカル内にあるアプリの更新情報を更新
scoop update

# 最新バージョンでないアプリがあるかをチェック
scoop status

# xxxxを更新
scoop update xxxx

# すべてのアプリを更新
scoop update *

# xxxxをアンインストール
scoop uninstall xxxx
```

## platformioの使い方

```

# プロジェクトのディレクトリを作る
# (一つのスケッチごとに一つのディレクトリ)
mkdir proj
cd proj

# ソースを置くディレクトリsrcを作る
mkdir src

# スケッチを作成する
# (テスト・スケッチとして以降にASCIITable.inoがある)
notepad src/main.ino

# 使うボードに対応したplatformio.iniを作る
# (内容については以降を参照のこと)
notepad platformio.ini

# スケッチをビルドする
# (最初の１回はライブラリ・ツールを自動的にダウンロードする)
pio run

# ボードをUSB接続して書き込む
pio run -t upload

```
platformio.iniの具体的な内容は「[arduinoフレームワーク用platformio.ini集](https://beta-notes.way-nifty.com/blog/2021/02/post-2b331d.html)」を参照のこと


その他の使い方
```

# build結果をクリアする
pio run -t clean

# キャッシュをクリアする
# (ツールやライブラリがダウンロードし直しになるので注意のこと)
rm -r .pio

# 環境を切り替えて書き込む
pio run -e f303 -t upload
pio run -e f103 -t upload 
```

環境を切り替えて書き込む場合のplatformio.iniは   
以下のように複数の環境[env:xxx]を持っていること： 
[]()   
platformio.ini
```

[env:f103]
platform = ststm32
board = nucleo_f103rb
framework = arduino
build_flags = -DNUCLEO_F103RB
monitor_speed = 115200
lib_ldf_mode = deep+

upload_protocol = stlink

#lib_deps = 

[env:f303]
platform = ststm32
board = nucleo_f303k8
framework = arduino
build_flags = -DNUCLEO_F303K8
monitor_speed = 115200
lib_ldf_mode = deep+

upload_protocol = stlink

#lib_deps = 
```
[env:xxxx]のxxxxの部分は環境名にあたり、
ダブらない任意の名前であること。

## テスト用スケッチ
src/ASCIItable.ino
```C++

/*
  ASCII table

  Prints out byte values in all possible formats:
  - as raw binary values
  - as ASCII-encoded decimal, hex, octal, and binary values

  For more on ASCII, see http://www.asciitable.com and http://en.wikipedia.org/wiki/ASCII

  The circuit: No external hardware needed.

  created 2006
  by Nicholas Zambetti <http://www.zambetti.com>
  modified 9 Apr 2012
  by Tom Igoe

  This example code is in the public domain.

  http://www.arduino.cc/en/Tutorial/ASCIITable
*/

void setup() {
  //Initialize serial and wait for port to open:
  Serial.begin(115200);
  while (!Serial) {
    ; // wait for serial port to connect. Needed for native USB port only
  }

  // prints title with ending line break
  Serial.println("ASCII Table ~ Character Map");
}

// first visible ASCIIcharacter '!' is number 33:
int thisByte = 33;
// you can also write ASCII characters in single quotes.
// for example, '!' is the same as 33, so you could also use this:
// int thisByte = '!';

void loop() {
  // prints value unaltered, i.e. the raw binary version of the byte.
  // The Serial Monitor interprets all bytes as ASCII, so 33, the first number,
  // will show up as '!'
  Serial.write(thisByte);

  Serial.print(", dec: ");
  // prints value as string as an ASCII-encoded decimal (base 10).
  // Decimal is the default format for Serial.print() and Serial.println(),
  // so no modifier is needed:
  Serial.print(thisByte);
  // But you can declare the modifier for decimal if you want to.
  // this also works if you uncomment it:

  // Serial.print(thisByte, DEC);


  Serial.print(", hex: ");
  // prints value as string in hexadecimal (base 16):
  Serial.print(thisByte, HEX);

  Serial.print(", oct: ");
  // prints value as string in octal (base 8);
  Serial.print(thisByte, OCT);

  Serial.print(", bin: ");
  // prints value as string in binary (base 2) also prints ending line break:
  Serial.println(thisByte, BIN);

  // if printed last visible character '~' or 126, stop:
  if (thisByte == 126) {    // you could also use if (thisByte == '~') {
    // This loop loops forever and does nothing
    while (true) {
      continue;
    }
  }
  // go on to the next character
  thisByte++;
}
```
本スケッチはarduinoのサンプルそのもの(bpsのみ変更)である。

## alias削除の永続化
インストール時に以下でalias削除したものは、PowerShellのセッションを閉じると無効になり復活する。
```
# 別の機能が割り当てられているaliasを削除する
del alias:curl
del alias:wget

```
したがって、削除を永続化するためにセッション起動時に自動実行されるPowerShellのスクリプト(Microsoft.PowerShell_profile.ps1)の
末尾に以下を追加する：
Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1   
(存在しない場合、新規に作成する)
```

del alias:curl
del alias:wget
```


## ビルド・エラー
Arduino-IDEのコンパイラと異なり、platformioのコンパイラは、関数定義の後方参照を許さないので、
この場合、関数の未定義エラーになる。したがって、Arduino-IDEでビルドできているソースでもエラーになることがある。このときの対応方法は、未定義エラーになる関数定義をプロトタイプ宣言としてソースの先頭に置き、後方参照を解消する。(関数定義の本体の位置はそのままで移動させる必要はない)


## 参考情報

[WindowsコマンドラインツールScoopのすすめ（基礎編）](https://qiita.com/talesleaves/items/0880bf31359715035a3c)  
[Windowsでパッケージ管理したいなら、先ずScoopより始めよ](https://www.jcsc.co.jp/tech_blog/archives/3)  
[PowerShell コンソール内で curl や wget が実行できないとお嘆きのあなたへ](https://devadjust.exblog.jp/22690878/)  

[PlatformIO M5ATOM on Windows10](https://beta-notes.way-nifty.com/blog/2020/10/post-7f9bc0.html)  
[TinyGO Install XIAO on Windows10](https://beta-notes.way-nifty.com/blog/2020/10/post-28fc78.html)  

M5Stack関連：  
・[PlatformIO M5Stack開発の現状](https://qiita.com/wararyo/items/fc3b90f72a18b24cf456)   
・https://github.com/m5stack/M5StickC-Plus.git  
・https://github.com/m5stack/M5StickC.git   
・https://github.com/m5stack/M5Stack.git   

micro:bit関連：  
・[micro:bit Arduino/MBED開発ツール(v2)(micro:bit-v2対応,linux版)](https://beta-notes.way-nifty.com/blog/2021/01/post-e6d91a.html)   
・[micro:bit v2 で遊ぶ](https://qiita.com/sat0ken/items/13bd03378c28b98a794e)   

platformio関連：
```

https://docs.platformio.org/en/latest/platforms/creating_board.html

Installation
1.Create boards directory in core_dir if it doesn’t exist.
2.Create myboard.json file in this boards directory.
3.Search available boards via pio boards command. You should see myboard board.
```

・[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  
・[Arduino-IDEとPlatformioのコンパイラーの挙動の違いについて](https://beta-notes.way-nifty.com/blog/2020/07/post-fbe8f7.html)   
・[ubuntu20.04をインストールする](https://beta-notes.way-nifty.com/blog/2020/10/post-fa7773.html)   
・[Advanced Scripting - Before/Pre and After/Post actions](https://docs.platformio.org/en/latest/projectconf/advanced_scripting.html#before-pre-and-after-post-actions)  

以上
