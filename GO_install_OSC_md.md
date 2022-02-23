
2020/4/19

GO Install OSC
# GO Install OSC

## 概要
プログラミング言語GOでOSC(Open Sound Control)を動かす。実行するプログラムはOSCライブラリのexamplesを流用している。
(ホストPCとしてはubuntuを想定している)

## OSCライブラリのインストール
```bash

go get github.com/hypebeast/go-osc

# 作業用ディレクトリを設定する
cd $GOPATH/src
mkdir OSC-go
cd OSC-go

```

## OSC受信プログラム
OSCRecv_demo.go
```go

package main

//import "github.com/hypebeast/go-osc/osc"
import (
	"fmt"
	"github.com/hypebeast/go-osc/osc"
)

func main() {
    addr := "0.0.0.0:8000"
    d := osc.NewStandardDispatcher()
 
    d.AddMsgHandler("/1/fader1", func(msg *osc.Message) {
        osc.PrintMessage(msg)
    })
 
    d.AddMsgHandler("/1/fader2", func(msg *osc.Message) {
        osc.PrintMessage(msg)
    })

    d.AddMsgHandler("/1/fader3", func(msg *osc.Message) {
        osc.PrintMessage(msg)
    })

    d.AddMsgHandler("/1/fader4", func(msg *osc.Message) {
        osc.PrintMessage(msg)
    })

    d.AddMsgHandler("/1/fader5", func(msg *osc.Message) {
        osc.PrintMessage(msg)
    })

    d.AddMsgHandler("/1/toggle1", func(msg *osc.Message) {
        osc.PrintMessage(msg)
    })

    d.AddMsgHandler("/1/toggle2", func(msg *osc.Message) {
        osc.PrintMessage(msg)
    })
    d.AddMsgHandler("/1/toggle3", func(msg *osc.Message) {
        osc.PrintMessage(msg)
    })
    d.AddMsgHandler("/1/toggle4", func(msg *osc.Message) {
        osc.PrintMessage(msg)
    })


    fmt.Println("Listening at:",addr)

    server := &osc.Server{
        Addr: addr,
        Dispatcher:d,
    }
    server.ListenAndServe()
}
```
入力ポートとして8000として、OSC受信する。

## OSC送信プログラム
OSCSend_demo.go 
```go

package main

import "github.com/hypebeast/go-osc/osc"

func main() {
    client := osc.NewClient("localhost", 8000)
    msg := osc.NewMessage("/1/fader1")
    msg.Append(int32(111))
    msg.Append(true)
    msg.Append("hello")
    client.Send(msg)
    msg1 := osc.NewMessage("/1/fader2")
    msg1.Append(float32(0.789))
    client.Send(msg1)
}
```
出力ポートとして8000として、OSC送信する。


## OSC message dump
任意のOSCメッセージを受信できるプログラム：

OSCdump.go
```go

package main

import (
	"bufio"
	"fmt"
	"net"
	"os"

	"github.com/hypebeast/go-osc/osc"
)

func main() {
	addr := "0.0.0.0:8000"
	server := &osc.Server{}
	conn, err := net.ListenPacket("udp", addr)
	if err != nil {
		fmt.Println("Couldn't listen: ", err)
	}
	defer conn.Close()

	fmt.Println("OSC msg Dump")
	fmt.Println("Press \"q\" to exit")

	go func() {
		fmt.Println("Start listening on", addr)

		for {
			packet, err := server.ReceivePacket(conn)
			if err != nil {
				fmt.Println("Server error: " + err.Error())
				os.Exit(1)
			}

			if packet != nil {
				switch packet.(type) {
				default:
					fmt.Println("Unknow packet type!")

				case *osc.Message:
					fmt.Printf("-- OSC Message: ")
					osc.PrintMessage(packet.(*osc.Message))

				case *osc.Bundle:
					fmt.Println("-- OSC Bundle:")
					bundle := packet.(*osc.Bundle)
					for i, message := range bundle.Messages {
						fmt.Printf("  -- OSC Message #%d: ", i+1)
						osc.PrintMessage(message)
					}
				}
			}
		}
	}()

	reader := bufio.NewReader(os.Stdin)

	for {
		c, err := reader.ReadByte()
		if err != nil {
			os.Exit(0)
		}

		if c == 'q' {
			os.Exit(0)
		}
	}
}
```
入力ポートとして8000として、OSC受信する。


## build
以下でビルドする：
```bash

go build OSCRecv_demo.go
go build OSCSend_demo.go 
go build OSCdump.go

```

## 実行と出力例
```bash

./OSCRecv_demo 
# TouchOSCからOSCを受信したときの出力
Listening at: 0.0.0.0:8000
/1/fader1 ,f 0.5864979
/1/fader2 ,f 0.53164554
/1/fader2 ,f 1
/1/fader2 ,f 0.98734176
/1/fader2 ,f 0.9704641
/1/fader3 ,f 0.46202534
...


./OSCdump 
# TouchOSCからOSCを受信したときの出力
OSC msg Dump
Press "q" to exit
Start listening on 0.0.0.0:8000
-- OSC Message: /1/fader2 ,f 0.6413502
-- OSC Message: /3/xy ,ff 0.33333334 0.32771537
-- OSC Message: /3/xy ,ff 0.6479401 0.6479401
-- OSC Message: /2/push6 ,f 1
-- OSC Message: /2/push6 ,f 0
-- OSC Message: /4/multitoggle/6/2 ,f 1
-- OSC Message: /4/multitoggle/6/6 ,f 1
-- OSC Message: /4/multitoggle/3/4 ,f 1
...

```


## 引用情報

[Open Sound Control (OSC) library for Golang. Implemented in pure Go.](https://github.com/hypebeast/go-osc)  

[TouchOSC](https://hexler.net/products/touchosc)  
[TouchOSC iPhone](https://apps.apple.com/app/touchosc/id288120394)  


以上
