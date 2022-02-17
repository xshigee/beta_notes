
2020/2/27+

ESP-WROOM-02 MicroPython TFT Shield
# ESP-WROOM-02 MicroPython TFT Shield

## 概要
「ESP-WROOM-02 Arduino互換ボード」(ESP8266)のMicroPythonに以下のTFT_Shieldを接続する。SDスロットを持っているのでSDもサポートする。Touchパネルもあるがハードウェアの制限もありサポートしない。
ここでは、linux環境でのインストール方法について説明する。   

[2.8 inch TFT Touch Shield v2.0](http://wiki.seeedstudio.com/2.8inch_TFT_Touch_Shield_v2.0/)   
(LCDコントローラ:ILI9341)  

## 配線表
実際にはシールドを刺すだけだが以下のように配線したことになる。

| Arduino_Pin | ESP8266_Pin | 
| :--: | :--: |
| D4 | IO2(TF_CS/SD_CS) |
| D5 | IO4(TFT_CS) |
| D6 | IO5(TFT_DC) |
| D7 | IO14(BACKLIGHT(Selectable))(NOT USED) |
| D10 | IO15(RST(dummy))|
| D11 | IO13（MOSI） |
| D12 | IO12（MISO） |
| D13 |	IO14（SCK） |

| Arduino_Pin | ESP8266_Pin | 
| :--: | :--: |
| A0 | TOUCH PANEL(NOT USED) |
| A1 | TOUCH PANEL(NOT USED) |
| A2 | TOUCH PANEL(NOT USED) |
| A3 | TOUCH PANEL(NOT USED) |

## 関連モジュールのインストール
以下の手順でインストールする：
```bash
# TFTモジュールのインストール
hg clone https://bitbucket.org/thesheep/micropython-ili9341
cd micropython-ili9341
# ili9341.pyにパッチを当てて、それをili9341_ESP8266.pyとする。
# 修正点は次項を参照のこと
ampy put ili9341_ESP8266.py

# SD card モジュールのインストール
wget https://raw.githubusercontent.com/micropython/micropython/master/drivers/sdcard/sdcard.py
ampy put sdcard.py

```

## ili9341_ESP8266.py
ili9341_ESP8266.py  
「#」でコメントにした行付近が修正点になる
```python

# ESP8266
import time
import ustruct
import framebuf

_COLUMN_SET = const(0x2a)
_PAGE_SET = const(0x2b)
_RAM_WRITE = const(0x2c)
_RAM_READ = const(0x2e)
_DISPLAY_ON = const(0x29)
_WAKE = const(0x11)
_LINE_SET = const(0x37)


def color565(r, g, b):
    return (r & 0xf8) << 8 | (g & 0xfc) << 3 | b >> 3


class ILI9341:
    """
    A simple driver for the ILI9341/ILI9340-based displays.


    >>> import ili9341
    >>> from machine import Pin, SPI
    >>> spi = SPI(miso=Pin(12), mosi=Pin(13, Pin.OUT), sck=Pin(14, Pin.OUT))
    >>> display = ili9341.ILI9341(spi, cs=Pin(0), dc=Pin(5), rst=Pin(4))
    >>> display.fill(ili9341.color565(0xff, 0x11, 0x22))
    >>> display.pixel(120, 160, 0)
    """

    width = 240
    height = 320

    def __init__(self, spi, cs, dc, rst):
        self.spi = spi
        self.cs = cs
        self.dc = dc
        self.rst = rst
        self.cs.init(self.cs.OUT, value=1)
        self.dc.init(self.dc.OUT, value=0)
        self.rst.init(self.rst.OUT, value=0)
        self.reset()
        self.init()
        self._scroll = 0

    def init(self):
        for command, data in (
            (0xef, b'\x03\x80\x02'),
            (0xcf, b'\x00\xc1\x30'),
            (0xed, b'\x64\x03\x12\x81'),
            (0xe8, b'\x85\x00\x78'),
            (0xcb, b'\x39\x2c\x00\x34\x02'),
            (0xf7, b'\x20'),
            (0xea, b'\x00\x00'),
            (0xc0, b'\x23'),  # Power Control 1, VRH[5:0]
            (0xc1, b'\x10'),  # Power Control 2, SAP[2:0], BT[3:0]
            (0xc5, b'\x3e\x28'),  # VCM Control 1
            (0xc7, b'\x86'),  # VCM Control 2
            (0x36, b'\x48'),  # Memory Access Control
            (0x3a, b'\x55'),  # Pixel Format
            (0xb1, b'\x00\x18'),  # FRMCTR1
            (0xb6, b'\x08\x82\x27'),  # Display Function Control
            (0xf2, b'\x00'),  # 3Gamma Function Disable
            (0x26, b'\x01'),  # Gamma Curve Selected
            (0xe0,  # Set Gamma
             b'\x0f\x31\x2b\x0c\x0e\x08\x4e\xf1\x37\x07\x10\x03\x0e\x09\x00'),
            (0xe1,  # Set Gamma
             b'\x00\x0e\x14\x03\x11\x07\x31\xc1\x48\x08\x0f\x0c\x31\x36\x0f'),
        ):
            self._write(command, data)
        self._write(_WAKE)
        time.sleep_ms(120)
        self._write(_DISPLAY_ON)

    def reset(self):
#        self.rst.low()
        self.rst.value(0)
        time.sleep_ms(50)
#        self.rst.high()
        self.rst.value(1)
        time.sleep_ms(50)

    def _write(self, command, data=None):
#        self.dc.low()
#        self.cs.low()
        self.dc.value(0)
        self.cs.value(0)
        self.spi.write(bytearray([command]))
#        self.cs.high()
        self.cs.value(1)
        if data is not None:
            self._data(data)

    def _data(self, data):
#        self.dc.high()
#        self.cs.low()
        self.dc.value(1)
        self.cs.value(0)
        self.spi.write(data)
#        self.cs.high()
        self.cs.value(1)

    def _block(self, x0, y0, x1, y1, data=None):
        self._write(_COLUMN_SET, ustruct.pack(">HH", x0, x1))
        self._write(_PAGE_SET, ustruct.pack(">HH", y0, y1))
        if data is None:
            return self._read(_RAM_READ, (x1 - x0 + 1) * (y1 - y0 + 1) * 3)
        self._write(_RAM_WRITE, data)

    def _read(self, command, count):
#        self.dc.low()
#        self.cs.low()
        self.dc.value(0)
        self.cs.value(0)
        self.spi.write(bytearray([command]))
        data = self.spi.read(count)
#        self.cs.high()
        self.cs.value(1)
        return data

    def pixel(self, x, y, color=None):
        if color is None:
            r, b, g = self._block(x, y, x, y)
            return color565(r, g, b)
        if not 0 <= x < self.width or not 0 <= y < self.height:
            return
        self._block(x, y, x, y, ustruct.pack(">H", color))

    def fill_rectangle(self, x, y, w, h, color):
        x = min(self.width - 1, max(0, x))
        y = min(self.height - 1, max(0, y))
        w = min(self.width - x, max(1, w))
        h = min(self.height - y, max(1, h))
        self._block(x, y, x + w - 1, y + h - 1, b'')
        chunks, rest = divmod(w * h, 512)
        if chunks:
            data = ustruct.pack(">H", color) * 512
            for count in range(chunks):
                self._data(data)
        data = ustruct.pack(">H", color) * rest
        self._data(data)

    def fill(self, color):
        self.fill_rectangle(0, 0, self.width, self.height, color)

    def char(self, char, x, y, color=0xffff, background=0x0000):
        buffer = bytearray(8)
        framebuffer = framebuf.FrameBuffer1(buffer, 8, 8)
        framebuffer.text(char, 0, 0)
        color = ustruct.pack(">H", color)
        background = ustruct.pack(">H", background)
        data = bytearray(2 * 8 * 8)
        for c, byte in enumerate(buffer):
            for r in range(8):
                if byte & (1 << r):
                    data[r * 8 * 2 + c * 2] = color[0]
                    data[r * 8 * 2 + c * 2 + 1] = color[1]
                else:
                    data[r * 8 * 2 + c * 2] = background[0]
                    data[r * 8 * 2 + c * 2 + 1] = background[1]
        self._block(x, y, x + 7, y + 7, data)

    def text(self, text, x, y, color=0xffff, background=0x0000, wrap=None,
             vwrap=None, clear_eol=False):
        if wrap is None:
            wrap = self.width - 8
        if vwrap is None:
            vwrap = self.height - 8
        tx = x
        ty = y

        def new_line():
            nonlocal tx, ty

            tx = x
            ty += 8
            if ty >= vwrap:
                ty = y

        for char in text:
            if char == '\n':
                if clear_eol and tx < wrap:
                    self.fill_rectangle(tx, ty, wrap - tx + 7, 8, background)
                new_line()
            else:
                if tx >= wrap:
                    new_line()
                self.char(char, tx, ty, color, background)
                tx += 8
        if clear_eol and tx < wrap:
            self.fill_rectangle(tx, ty, wrap - tx + 7, 8, background)

    def scroll(self, dy=None):
        if dy is None:
            return self._scroll
        self._scroll = (self._scroll + dy) % self.height
        self._write(_LINE_SET, ustruct.pack('>H', self._scroll))

```

## 動作確認用スクリプト

TFT_SD_test_ESP8266.py
```python
# TFT Touch Shield(with SD)
# (does not support Touch)
# setup display module
import ili9341_ESP8266
color565 = ili9341_ESP8266.color565
from machine import Pin, SPI
spi = SPI(miso=Pin(12), mosi=Pin(13, Pin.OUT), sck=Pin(14, Pin.OUT))
#spi=SPI(1)
display = ili9341_ESP8266.ILI9341(spi, cs=Pin(4), dc=Pin(5), rst=Pin(15))
# setup SDCard module
import machine, sdcard, os
sd = sdcard.SDCard(spi, Pin(2))
os.mount(sd, '/sd')
# setup path
import sys
sys.path.append('/sd')
sys.path.append('/sd/lib')
# display test
display.fill(color565(0xff, 0x11, 0x22))
display.fill(color565(0xff, 0xff, 0xff))
display.fill(color565(0x00, 0x00, 0x00))
display.pixel(120, 160, color565(0xff,0,0))
display.pixel(121, 161, color565(0,0,0xff))
display.pixel(123, 163, color565(0,0xff,0))
display.text('Hello World!',0,0,color=color565(0,0xff,00))
display.text('0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz{}()+*<>?_',100,200,color=color565(0,0,0xff))
display.fill_rectangle(100,150,20, 50, color565(0xff,0,0))
# SD Card test
display.fill(color565(0x00, 0x00, 0xff))
display.text(str(sys.path),0,0,color565(0xff,0xff,0xff),clear_eol=True)
display.text(str(os.listdir('/sd')),0,0,color565(0xff,0xff,0xff),clear_eol=True)
```

## 実行

```bash

picocom /dev/ttyUSB0 -b115200

>>> 
MPY: soft reboot
MicroPython v1.12 on 2019-12-20; ESP module with ESP8266
Type "help()" for more information.
>>> 

>>> import TFT_SD_test_ESP8266

```

## 参考情報  

[ESP-WROOM-02 Arduino互換ボード](https://www.switch-science.com/catalog/2620/)   
[lvidarte/esp8266](https://github.com/lvidarte/esp8266/wiki/MicroPython:-Examples)   
[MicroPython tutorial for ESP826](https://docs.micropython.org/en/v1.9.2/esp8266/esp8266/tutorial/index.html)   
[ampyを用いたMicroPythonのファイル操作とプログラム実行](https://blog.goediy.com/?p=335)　　　

[2.8 inch TFT Touch Shield v2.0](http://wiki.seeedstudio.com/2.8inch_TFT_Touch_Shield_v2.0/)   

他のILI9341サポートモジュール：   
[Micropython Driver for ILI9341 display](https://github.com/jeffmer/micropython-ili9341.git)  
[Micropython TFT Display Driver for ILI9341 Chipset](https://github.com/tkurbad/micropython-ili9341.git)  
Arduino用ライブラリー：  
[Seeed-Studio/TFT_Touch_Shield_V2/archive/master.zip](https://github.com/Seeed-Studio/TFT_Touch_Shield_V2/archive/master.zip)   


以上

