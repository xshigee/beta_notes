
2020/5/4  
改良コード追加  

2020/5/3++  
初版  

PlatformIO Longan Nano Movies
# PlatformIO Longan Nano Movies

## 概要
Longan-NanoのLCDに動画を表示する。SDKのサンプルコードを流用して動画を再生してみる。
(ホストPCとしてはubuntuを想定している)


## PlatformIOのインストール
```bash

python3 -m venv pio_env
source pio_env/bin/activate

pip3 install platformio

インストール後も、本ツールを使用する場合
同じディレクトリで以下を実行する：  
source pio_env/bin/activate
# 「source」は、「.」でも良い

```
## 準備
以下を実行して、udevのrulesを登録する：
```bash


curl -fsSL https://raw.githubusercontent.com/platformio/platformio-core/master/scripts/99-platformio-udev.rules | sudo tee /etc/udev/rules.d/99-platformio-udev.rules

sudo udevadm control --reload-rules

sudo usermod -a -G dialout $USER
sudo usermod -a -G plugdev $USER

```

## サンプル・コード
このサンプル・コードを利用してLCDに動画を表示する。
本プログラム自身には手を加えず、動画データであるbmp.binを差し替えることで任意の動画を表示する。
以下の手順で、サンプル・コードをビルド＆実行してみる。
```bash

$ cd pio_ws
$ . pio_env/bin/activate

git clone https://github.com/sipeed/Longan_GD32VF_examples.git
cd Longan_GD32VF_examples

#　32GB以下のmicroSDに以下の２つのファイルをコピーして、
# そのmicroSDをボードのTFスロットに刺しておく。
$ ls put_into_tf_card/
bmp.bin  logo.bin

cd Longan_GD32VF_examples/gd32v_lcd

pio run

# ボードをPCに接続して
# 以下の手順で書き込みモード(DFUモード)にする：
# (1)「BOOT」ボタンと「RESET」ボタンも同時に押す。
# (2)１秒くらいして「RESET」ボタンを離す。
# (3)その後、「BOOT」も離す。
pio run -t upload

# 以上で書き込みが終わり、LCDにデモのアニメーションが表示される。

```
次で、差し替えるためのbmp.binの作成方法を説明する。


## 動画データbmp.binの作成
ツールのインストールとパッチ：
```bash

sudo apt-get install ffmpeg

mkdir BMP_ws
cd BMP_ws
curl -OL http://dl.sipeed.com/LONGAN/Nano/Firmware/badapple_demo_tools/tools_bmp2hex.zip
unzip tools_bmp2hex.zip
# unzipで以下のファイルに解凍される：
# bmp2hex.py  genhex.py  rename.py
# genhex.pyは以下のように修正する：
# (rename.pyは使用しない)

```

以下のように修正(patch)する：  
genhex.py
```python

import os
import sys # for cli args
args = sys.argv # get args

if os.path.exists('bmp.bin'):
    os.remove('bmp.bin')

num = 1
while num <= int(args[1]):
#while num <= 2189:
    #os.system("python.exe bmp2hex.py -kbin {0}.bmp".format(num))
    os.system("python3 bmp2hex.py -kbin {:04d}.bmp".format(num))
    num += 1

```
素材となる動画ファイルをダウンロード(入手)する：  
(素材となる動画ファイルは任意である)  
```bash

#以下のうち、１つを実行して素材をダウンロードする：
wget https://archive.org/download/BigBuckBunny_310/big_buck_bunny_640.mp4
wget https://archive.org/download/Sintel/sintel-2048-stereo.mp4
wget http://ftp.nluug.nl/pub/graphics/blender/demo/movies/ToS/tears_of_steel_720p.mov

```
以下の手順で動画ファイルをbmp.binに変換する：  
(ここでは素材例としてbig_buck_bunny_640.mp4を使う)
```bash

cd BMP_ws
# *.bmpがあれば削除する
rm *.bmp
# 以下を実行して、動画のフレームをBMPファイルに変換する：
#(複数のBMPファイルができる)
ffmpeg -ss 10 -t 300 -i big_buck_bunny_640.mp4 -r 12 -s 160x80 -vcodec bmp %04d.bmp
# 上の例では「 -ss 10 -t 300」で頭出し10秒後、再生時間を300秒を指定しているが任意である。
# 留意点：数多くのBMPファイルを作るので時間がかかる
# 実行が完了したら、<フレーム番号>.bmpの出来上がるので、最大の番号を確認する。
# 例えば、それが　9999 ならば
# 以下を実行する：
python3 genhex.py 9999
# 各bmpファイルを結合して,bmp.binを作成するので
# (時間がかかるので)終わるまで待つ。
```

以上で作成したbmp.binをLongan-Nanoに挿しているmicroSDにコピーする。
ボードを再起動すると、ロゴ画面に続いて、bmp.binに変換した動画が再生される。

## 補足
(1)動画の最大フレーム数が大きいと最後まで再生されないことがある。  
そのときは、main.cの以下の部分のフレーム数の設定を以下のように変更する。  
行番号87「for (int i=0; i<2189;i++)」→「for (int i=0; i<9999;i++)」

最大フレーム数よりも大きい値を設定しておくとプログラムをいちいち変更せずに済むので便利だと思う。
（ただし、最終的にエラーで止まるので、オリジナルのように繰り返し実行しなくなるが
リセットボタンを押せば、再実行するので、特に問題ないと思う）

(2)デモプログラムの流用なので時間軸の制御はしていない。  
設定としては、fpsは12だが、実際には、それよりも多少速く再生されるようだ。  

(3)最終フレームの画像が半分になって上下に表示されるバグがあるようだ。  
回避策として、最終フレームとして全画面黒一色のものを追加して、bmp.binを作ることで回避できそうだ。

## 改良コード
ロゴ画面を表示しているときにBOOT0ボタンを押し続けると、別のファイル(bmp2.bin)を選択して
動画を表示する機能を追加した。  

以下にmain.cの差分を記載する：  
(1)main(void)の中のGPIO初期化部分を以下のように変更する：  
(「PA8を入力モードに初期化する」を追加する)  
```c

    rcu_periph_clock_enable(RCU_GPIOA);
    rcu_periph_clock_enable(RCU_GPIOC);
    // init for LED_R(PC13)
    gpio_init(GPIOC, GPIO_MODE_OUT_PP, GPIO_OSPEED_50MHZ, GPIO_PIN_13);
    // init for LED_G(PA1),LED_B(PA2)
    gpio_init(GPIOA, GPIO_MODE_OUT_PP, GPIO_OSPEED_50MHZ, GPIO_PIN_1|GPIO_PIN_2);
    // configure BOOT0 button pin(PA8) as input
    gpio_init(GPIOA, GPIO_MODE_IN_FLOATING, GPIO_OSPEED_50MHZ, GPIO_PIN_8);

    init_uart0();
```
(2)main(void)の"bmp.bin"をopenする部分を以下のように変更する：
```c
            f_close(&fil);

            // check whether the BOOT0 button is pressed
            if(RESET ==  gpio_input_bit_get(GPIOA, GPIO_PIN_8)){
               // select default
               fr = f_open(&fil, "bmp.bin", FA_READ);
            } else {
               // select 2nd option
               fr = f_open(&fil, "bmp2.bin", FA_READ);
            };

            //fr = f_open(&fil, "bmp.bin", FA_READ);

```

## 参考情報

ffmpeg:  
[ffmpegの使い方](http://tech.ckme.co.jp/ffmpeg.shtml)  
[それFFmpegで出来るよ！](https://qiita.com/cha84rakanal/items/e84fe4eb6fbe2ae13fd8)  
[ffmpeg使い方のまとめ](https://yoshipc.net/how-to-use-ffmpeg/)  
[Re:ゼロから始めるFFmpeg](https://nyanshiba.hatenablog.com/entry/2018/02/03/071256)  
[FFmpegで秒数を指定して動画を切り出すワンライナー](https://hori-ryota.com/blog/ffmpeg-mp4-cut/)  

[Ｓｉｐｅｅｄ　Ｌｏｎｇａｎ　Ｎａｎｏ　ＲＩＳＣ－Ｖ　ＧＤ３２ＶＦ１０３ＣＢＴ６開発ボード](http://akizukidenshi.com/catalog/g/gK-14678/)   
■主な仕様  
・CPU：GD32VF103CBT6  
・メモリ：128KB Flash/32KB SRAM  
・160×80ドット、0.96インチのフルカラーIPS液晶  
・TFスロット(microSDスロット)  

[Longan Nano PINOUT](https://longan.sipeed.com/assets/longan_nano_pinout_v1.1.0_w5676_h4000_large.png)    
[download Longan Nano Datasheet](https://cn.dl.sipeed.com/LONGAN/Nano/Spec/Sipeed%20longan%20nano%20Datasheet%20V1.0.pdf)

[LonganNanoで「RISC-Vちょっとできる」になろう！（ハードウェア編）](https://phillowocompile.blogspot.com/2019/11/longannanorisc-v.html)  
[GD32VF103 RISC-V](https://mecrisp-stellaris-folkdoc.sourceforge.io/gd32vf103.html)   

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

[Longan nano で Hello World!](https://qiita.com/myston/items/d7366021f75dce000c3b)   
[Sipeed Longan Nanoで文字を表示してみる](http://kyoro205.blog.fc2.com/blog-entry-667.html)   
・図形描画の関数の説明がある  
[Sipeed Longan Nano 用の、FONTX2対応LCD描画ライブラリとサンプルプログラム](https://github.com/takamame0205/LonganNanoLCD)  

[開発ツールPlatformIOでFT232Hボードをデバッガーとして使用する(Longan-Nano版)](https://beta-notes.way-nifty.com/blog/2020/05/post-5c077f.html)  

[Sipeed Longan Nano（その4）](http://nekomemo2.site/?eid=1852)  

以上

