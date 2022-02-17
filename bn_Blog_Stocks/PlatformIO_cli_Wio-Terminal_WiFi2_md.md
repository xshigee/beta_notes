
2020/7/11

PlatformIO cli Wio Terminal WiFi2
# PlatformIO cli Wio Terminal WiFi2

## 概要
Wio-TerminalでWiFiで使う(その２:
STARTWARS,WEB_ACCESS)      
本記事は、[[Wio-TerminalでWiFiで使う(Seeeduino-Wio-Terminal/Arduino版)](https://beta-notes.way-nifty.com/blog/2020/07/post-c6b11e.html)]の続きになる。   
(ホストPCとしてはubuntuを想定している)


## wifi_starwars
ASCIIアートでstarwarsのアニメを表示するスケッチ：

src/wifi_starwars.ino
```c++

#include <AtWiFi.h>

const char* ssid = "your_ssid";
const char* password = "yours_passwd";

// Use WiFiClient class to create TCP connections
WiFiClient client;
 
void setup() {
    Serial.begin(115200);
    while(!Serial); // Wait for Serial to be ready
    delay(1000);
 
    // Set WiFi to station mode and disconnect from an AP if it was previously connected
    WiFi.mode(WIFI_STA);
    WiFi.disconnect();
    delay(2000);
 
    WiFi.begin(ssid, password);
 
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.println("Connecting to WiFi..");
    }
    Serial.println("Connected to the WiFi network");
    Serial.print("IP Address: ");
    Serial.println (WiFi.localIP()); // prints out the device's IP address

    // set url for telnet
    const char* host = "towel.blinkenlights.nl";
    const uint16_t port = 23;

    Serial.print("Connecting to ");
    Serial.println(host);
  
    while (!client.connect(host, port)) {
        Serial.println("Connection failed.");
        Serial.println("Waiting 5 seconds before retrying...");
        delay(5000);
    }
}

void loop() {
    while (!client.available()) {
        delay(1); //delay 1 msec
    }

    // no buffering for ASCII Art Animation
    while (client.available()) {
        char c = client.read();
        Serial.write(c);
    }
/*********** 
    if (client.available() > 0) {
        String line = client.readString(); // Read from the server response
        Serial.print(line);
    }
************/
}
```

出力例：
```

picocom /dev/ttyACM0 -b115200

Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connected to the WiFi network
IP Address: 192.168.0.16

#以下のようにstarwarsのASCIIアートのアニメが表示される
                                                      
      Original Work   : Simon Jansen ( http://www.asciimation.co.nz/ )   
      Telnetification : Sten Spans ( http://blinkenlights.nl/ )          
      Terminal Tricks : Mike Edwards (pf-asciimation@mirkwood.net)       
                                                                         
      The hard work was done by Simon and Mike,                          
      I just placed it online in a different format.                     
                                                                         
      So long And Thanks for all the fish                                
                                                                         
      Sten (I just need a Hug)  

...
<省略>
...

```
一部、以下のようなATコマンドのレスポンス・ヘッダーが表示されることがある。
wifi関係のライブラリにバグがあるようだ。

```

+IPD,4,988,94.142.241.111,23:__|_____|___________|__|____________|_||____
+IPD,4,1460,94.142.241.111,23:
+IPD,4,1460,94.142.241.111,23:               |(I/ /][][\|   {}/   {}    \[][][] 

```


## wifi_access_web
webにアクセスするスケッチ：

src/wifi_accesss_web.ino
```c++


#include <AtWiFi.h>

// change your server IP
#define TargetIP "192.168.0.33"
#define DefaultPort 80

const char* ssid = "your_ssid";
const char* password = "your_passwd";
 
void setup() {
    Serial.begin(115200);
    while(!Serial); // Wait for Serial to be ready
    delay(1000);
 
    // Set WiFi to station mode and disconnect from an AP if it was previously connected
    WiFi.mode(WIFI_STA);
    WiFi.disconnect();
    delay(2000);
 
    WiFi.begin(ssid, password);
 
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.println("Connecting to WiFi..");
    }
    Serial.println("Connected to the WiFi network");
    Serial.print("IP Address: ");
    Serial.println (WiFi.localIP()); // prints out the device's IP address
}
 
 
void loop() {
    const uint16_t port = DefaultPort; // Default port
    const char* host = TargetIP;  // Target Server IP Address
 
    Serial.print("Connecting to ");
    Serial.println(host);
 
    // Use WiFiClient class to create TCP connections
    WiFiClient client;
 
    if (!client.connect(host, port)) {
        Serial.println("Connection failed.");
        Serial.println("Waiting 5 seconds before retrying...");
        delay(5000);
        return;
    }
 
    // This will send a request to the server
    //uncomment this line to send an arbitrary string to the server
    //client.print("Send this data to the server");
    //uncomment this line to send a basic document request to the server
    client.print("GET /index.html HTTP/1.1\n\n"); // sending HTTP GET request

    int maxloops = 0;
 
    //wait for the server's reply to become available
    while (!client.available() && maxloops < 1000) {
        maxloops++;
        delay(1); //delay 1 msec
    }
    if (client.available() > 0) {
        //read back one line from the server
        String line = client.readString(); // Read from the server response
        // Proceed various line-endings
        line.replace("\r\n", "\n");
        line.replace('\r', '\n');
        line.replace("\n", "\r\n");
        Serial.println(line);
    } else {
        Serial.println("client.available() timed out ");
    }
 
    Serial.println("Closing connection.");
    client.stop();
 
    Serial.println("Waiting 5 seconds before restarting...");
    delay(5000);
}
```
以下を自分の環境に合わせて変更すること：  
```
// change your server IP
#define TargetIP "192.168.0.33"
#define DefaultPort 80

const char* ssid = "your_ssid";
const char* password = "your_passwd";
```
上の例では、アクセスするwebサーバーのIPを"192.168.0.33"に設定している。

スケッチを起動する前に以下のコマンドでwebサーバーを起動しておく：
```

sudo python3 -m http.<html>
```

サーバーのindex.htmlは以下にする：
```html

<html>
<body>
Hello World!
</body>
</html>
```

出力例：
```

picocom /dev/ttyACM0 -b115200

Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connecting to WiFi..
Connected to the WiFi network
IP Address: 192.168.0.14
Connecting to 192.168.0.33
HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/3.5.2
Date: Sat, 11 Jul 2020 06:11:53 GMT
Content-type: text/html
Content-Length: 43
Last-Modified: Sat, 11 Jul 2020 06:09:01 GMT

<html>
<body>
Hello World!
</body>
</html>

Closing connection.

```

## wifi_access_https_web
httpsサーバーにアクセスするスケッチ：  

src/wifi_access_https_web.ino
```c++


// define one of them for web access
#define EXAMPLE_COM
//#define NIFTY_COM
//#define GOOGLE_COM

#include <WiFiClientSecure.h>

const char* ssid     = "your_ssid";   // your network SSID
const char* password = "your_passwd"; // your network password

#ifdef GOOGLE_COM
const char* server = "www.google.com";  // Server URL 
const char* test_root_ca = \
"-----BEGIN CERTIFICATE-----\n"
"MIIESjCCAzKgAwIBAgINAeO0mqGNiqmBJWlQuDANBgkqhkiG9w0BAQsFADBMMSAw\n"
"HgYDVQQLExdHbG9iYWxTaWduIFJvb3QgQ0EgLSBSMjETMBEGA1UEChMKR2xvYmFs\n"
"U2lnbjETMBEGA1UEAxMKR2xvYmFsU2lnbjAeFw0xNzA2MTUwMDAwNDJaFw0yMTEy\n"
"MTUwMDAwNDJaMEIxCzAJBgNVBAYTAlVTMR4wHAYDVQQKExVHb29nbGUgVHJ1c3Qg\n"
"U2VydmljZXMxEzARBgNVBAMTCkdUUyBDQSAxTzEwggEiMA0GCSqGSIb3DQEBAQUA\n"
"A4IBDwAwggEKAoIBAQDQGM9F1IvN05zkQO9+tN1pIRvJzzyOTHW5DzEZhD2ePCnv\n"
"UA0Qk28FgICfKqC9EksC4T2fWBYk/jCfC3R3VZMdS/dN4ZKCEPZRrAzDsiKUDzRr\n"
"mBBJ5wudgzndIMYcLe/RGGFl5yODIKgjEv/SJH/UL+dEaltN11BmsK+eQmMF++Ac\n"
"xGNhr59qM/9il71I2dN8FGfcddwuaej4bXhp0LcQBbjxMcI7JP0aM3T4I+DsaxmK\n"
"FsbjzaTNC9uzpFlgOIg7rR25xoynUxv8vNmkq7zdPGHXkxWY7oG9j+JkRyBABk7X\n"
"rJfoucBZEqFJJSPk7XA0LKW0Y3z5oz2D0c1tJKwHAgMBAAGjggEzMIIBLzAOBgNV\n"
"HQ8BAf8EBAMCAYYwHQYDVR0lBBYwFAYIKwYBBQUHAwEGCCsGAQUFBwMCMBIGA1Ud\n"
"EwEB/wQIMAYBAf8CAQAwHQYDVR0OBBYEFJjR+G4Q68+b7GCfGJAboOt9Cf0rMB8G\n"
"A1UdIwQYMBaAFJviB1dnHB7AagbeWbSaLd/cGYYuMDUGCCsGAQUFBwEBBCkwJzAl\n"
"BggrBgEFBQcwAYYZaHR0cDovL29jc3AucGtpLmdvb2cvZ3NyMjAyBgNVHR8EKzAp\n"
"MCegJaAjhiFodHRwOi8vY3JsLnBraS5nb29nL2dzcjIvZ3NyMi5jcmwwPwYDVR0g\n"
"BDgwNjA0BgZngQwBAgIwKjAoBggrBgEFBQcCARYcaHR0cHM6Ly9wa2kuZ29vZy9y\n"
"ZXBvc2l0b3J5LzANBgkqhkiG9w0BAQsFAAOCAQEAGoA+Nnn78y6pRjd9XlQWNa7H\n"
"TgiZ/r3RNGkmUmYHPQq6Scti9PEajvwRT2iWTHQr02fesqOqBY2ETUwgZQ+lltoN\n"
"FvhsO9tvBCOIazpswWC9aJ9xju4tWDQH8NVU6YZZ/XteDSGU9YzJqPjY8q3MDxrz\n"
"mqepBCf5o8mw/wJ4a2G6xzUr6Fb6T8McDO22PLRL6u3M4Tzs3A2M1j6bykJYi8wW\n"
"IRdAvKLWZu/axBVbzYmqmwkm5zLSDW5nIAJbELCQCZwMH56t2Dvqofxs6BBcCFIZ\n"
"USpxu6x6td0V7SvJCCosirSmIatj/9dSSVDQibet8q/7UK4v4ZUN80atnZz1yg==\n"
"-----END CERTIFICATE-----\n";
#endif

#ifdef NIFTY_COM
const char*  server = "www.nifty.com";  // Server URL 
const char* test_root_ca = \
"-----BEGIN CERTIFICATE-----\n"
"MIIEizCCA3OgAwIBAgIQBUb+GCP34ZQdo5/OFMRhczANBgkqhkiG9w0BAQsFADBh\n"
"MQswCQYDVQQGEwJVUzEVMBMGA1UEChMMRGlnaUNlcnQgSW5jMRkwFwYDVQQLExB3\n"
"d3cuZGlnaWNlcnQuY29tMSAwHgYDVQQDExdEaWdpQ2VydCBHbG9iYWwgUm9vdCBD\n"
"QTAeFw0xNzExMDYxMjIzNDVaFw0yNzExMDYxMjIzNDVaMF4xCzAJBgNVBAYTAlVT\n"
"MRUwEwYDVQQKEwxEaWdpQ2VydCBJbmMxGTAXBgNVBAsTEHd3dy5kaWdpY2VydC5j\n"
"b20xHTAbBgNVBAMTFEdlb1RydXN0IFJTQSBDQSAyMDE4MIIBIjANBgkqhkiG9w0B\n"
"AQEFAAOCAQ8AMIIBCgKCAQEAv4rRY03hGOqHXegWPI9/tr6HFzekDPgxP59FVEAh\n"
"150Hm8oDI0q9m+2FAmM/n4W57Cjv8oYi2/hNVEHFtEJ/zzMXAQ6CkFLTxzSkwaEB\n"
"2jKgQK0fWeQz/KDDlqxobNPomXOMJhB3y7c/OTLo0lko7geG4gk7hfiqafapa59Y\n"
"rXLIW4dmrgjgdPstU0Nigz2PhUwRl9we/FAwuIMIMl5cXMThdSBK66XWdS3cLX18\n"
"4ND+fHWhTkAChJrZDVouoKzzNYoq6tZaWmyOLKv23v14RyZ5eqoi6qnmcRID0/i6\n"
"U9J5nL1krPYbY7tNjzgC+PBXXcWqJVoMXcUw/iBTGWzpwwIDAQABo4IBQDCCATww\n"
"HQYDVR0OBBYEFJBY/7CcdahRVHex7fKjQxY4nmzFMB8GA1UdIwQYMBaAFAPeUDVW\n"
"0Uy7ZvCj4hsbw5eyPdFVMA4GA1UdDwEB/wQEAwIBhjAdBgNVHSUEFjAUBggrBgEF\n"
"BQcDAQYIKwYBBQUHAwIwEgYDVR0TAQH/BAgwBgEB/wIBADA0BggrBgEFBQcBAQQo\n"
"MCYwJAYIKwYBBQUHMAGGGGh0dHA6Ly9vY3NwLmRpZ2ljZXJ0LmNvbTBCBgNVHR8E\n"
"OzA5MDegNaAzhjFodHRwOi8vY3JsMy5kaWdpY2VydC5jb20vRGlnaUNlcnRHbG9i\n"
"YWxSb290Q0EuY3JsMD0GA1UdIAQ2MDQwMgYEVR0gADAqMCgGCCsGAQUFBwIBFhxo\n"
"dHRwczovL3d3dy5kaWdpY2VydC5jb20vQ1BTMA0GCSqGSIb3DQEBCwUAA4IBAQAw\n"
"8YdVPYQI/C5earp80s3VLOO+AtpdiXft9OlWwJLwKlUtRfccKj8QW/Pp4b7h6QAl\n"
"ufejwQMb455OjpIbCZVS+awY/R8pAYsXCnM09GcSVe4ivMswyoCZP/vPEn/LPRhH\n"
"hdgUPk8MlD979RGoUWz7qGAwqJChi28uRds3thx+vRZZIbEyZ62No0tJPzsSGSz8\n"
"nQ//jP8BIwrzBAUH5WcBAbmvgWfrKcuv+PyGPqRcc4T55TlzrBnzAzZ3oClo9fTv\n"
"O9PuiHMKrC6V6mgi0s2sa/gbXlPCD9Z24XUMxJElwIVTDuKB0Q4YMMlnpN/QChJ4\n"
"B0AFsQ+DU0NCO+f78Xf7\n"
"-----END CERTIFICATE-----\n";
#endif

#ifdef EXAMPLE_COM
const char*  server = "www.example.com";  // Server URL
const char* test_root_ca = \                            
"-----BEGIN CERTIFICATE-----\n"
"MIIDrzCCApegAwIBAgIQCDvgVpBCRrGhdWrJWZHHSjANBgkqhkiG9w0BAQUFADBh\n"
"MQswCQYDVQQGEwJVUzEVMBMGA1UEChMMRGlnaUNlcnQgSW5jMRkwFwYDVQQLExB3\n"
"d3cuZGlnaWNlcnQuY29tMSAwHgYDVQQDExdEaWdpQ2VydCBHbG9iYWwgUm9vdCBD\n"
"QTAeFw0wNjExMTAwMDAwMDBaFw0zMTExMTAwMDAwMDBaMGExCzAJBgNVBAYTAlVT\n"
"MRUwEwYDVQQKEwxEaWdpQ2VydCBJbmMxGTAXBgNVBAsTEHd3dy5kaWdpY2VydC5j\n"
"b20xIDAeBgNVBAMTF0RpZ2lDZXJ0IEdsb2JhbCBSb290IENBMIIBIjANBgkqhkiG\n"
"9w0BAQEFAAOCAQ8AMIIBCgKCAQEA4jvhEXLeqKTTo1eqUKKPC3eQyaKl7hLOllsB\n"
"CSDMAZOnTjC3U/dDxGkAV53ijSLdhwZAAIEJzs4bg7/fzTtxRuLWZscFs3YnFo97\n"
"nh6Vfe63SKMI2tavegw5BmV/Sl0fvBf4q77uKNd0f3p4mVmFaG5cIzJLv07A6Fpt\n"
"43C/dxC//AH2hdmoRBBYMql1GNXRor5H4idq9Joz+EkIYIvUX7Q6hL+hqkpMfT7P\n"
"T19sdl6gSzeRntwi5m3OFBqOasv+zbMUZBfHWymeMr/y7vrTC0LUq7dBMtoM1O/4\n"
"gdW7jVg/tRvoSSiicNoxBN33shbyTApOB6jtSj1etX+jkMOvJwIDAQABo2MwYTAO\n"
"BgNVHQ8BAf8EBAMCAYYwDwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQUA95QNVbR\n"
"TLtm8KPiGxvDl7I90VUwHwYDVR0jBBgwFoAUA95QNVbRTLtm8KPiGxvDl7I90VUw\n"
"DQYJKoZIhvcNAQEFBQADggEBAMucN6pIExIK+t1EnE9SsPTfrgT1eXkIoyQY/Esr\n"
"hMAtudXH/vTBH1jLuG2cenTnmCmrEbXjcKChzUyImZOMkXDiqw8cvpOp/2PV5Adg\n"
"06O/nVsJ8dWO41P0jmP6P6fbtGbfYmbW0W5BjfIttep3Sp+dWOIrWcBAI+0tKIJF\n"
"PnlUkiaY4IBIqDfv8NZ5YBberOgOzW6sRBc4L0na4UU+Krk2U886UAb3LujEV0ls\n"
"YSEY1QSteDwsOoBrp+uvFRTp2InBuThs4pFsiv9kuXclVzDAGySj4dzp30d8tbQk\n"
"CAUw7C29C79Fv1C5qfPrmAESrciIxpg0X40KPMbp1ZWVbd4=\n"
"-----END CERTIFICATE-----\n";
#endif
 
// You can use x.509 client certificates if you want
//const char* test_client_key = "";   //to verify the client
//const char* test_client_cert = "";  //to verify the client
 
WiFiClientSecure client;
 
void setup() {
    //Initialize serial and wait for port to open:
    Serial.begin(115200);
    while(!Serial); // Wait for Serial to be ready
    delay(1000);
 
    Serial.print("Attempting to connect to SSID: ");
    Serial.println(ssid);
    WiFi.begin(ssid, password);
 
    // attempt to connect to Wifi network:
    while (WiFi.status() != WL_CONNECTED) {
        Serial.print(".");
        // wait 1 second for re-trying
        delay(1000);
    }
    Serial.print("Connected to ");
    Serial.println(ssid);
 
    client.setCACert(test_root_ca);
    //client.setCertificate(test_client_key); // for client verification
    //client.setPrivateKey(test_client_cert); // for client verification
 
    Serial.println("\nStarting connection to server...");
    if (!client.connect(server, 443)) {
        Serial.println("Connection failed!");
    } else {
        Serial.println("Connected to server!");
        // Make a HTTP request:
#ifdef EXAMPLE_COM
        client.println("GET https://www.example.com HTTP/1.0");
        client.println("Host: www.example.com");
#endif
#ifdef NIFTY_COM
        client.println("GET https://www.nifty.com HTTP/1.0");
        client.println("Host: www.nifty.com");
#endif
#ifdef GOOGLE_COM
        client.println("GET https://www.google.com HTTP/1.0");
        client.println("Host: www.google.com");
#endif
        client.println("Connection: close");
        client.println();
 
        while (client.connected()) {
            String line = client.readStringUntil('\n');
            if (line == "\r") {
                Serial.println("headers received");
                break;
            }
        }
        // if there are incoming bytes available
        // from the server, read them and print them:
        while (client.available()) {
            char c = client.read();
            if (c == '\n') {
                Serial.write('\r');
            }
            Serial.write(c);
        }
        client.stop();
    }
}
 
void loop() {
    // do nothing
}
```

httpsでアクセスするために証明書(CA)が必要になるが
それは以下の手順で作成できる。できたCAはソースに埋め込む。
```

// how to get the root CA for www.example.com
sudo apt update
sudo apt install openssl
openssl s_client -showcerts -verify 5 -connect www.example.com:443 < /dev/null

//get the following string as CA from ouput text
-----BEGIN CERTIFICATE-----
...
＜省略＞
...
-----END CERTIFICATE-----

```
「www.example.com」の部分は、アクセスしたいサーバーのurlに変更してCAを生成する。

出力例：
```

picocom /dev/ttyACM0-b115200

Terminal ready
Attempting to connect to SSID: xxxxxxxxxxxx
.............Connected to xxxxxxxxxxxx

Starting connection to server...
Connected to server!
headers received
<!doctype html>
<html>
<head>
    <title>Example Domain</title>

    <meta charset="utf-8" />
    <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style type="text/css">
    body {
        background-color: #f0f0f2;
        margin: 0;
        padding: 0;
        font-family: -apple-system, system-ui, BlinkMacSystemFont, "Segoe UI", "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
        
    }
    div {
        width: 600px;
        margin: 5em auto;
        padding: 2em;
        background-color: #fdfdff;
        border-radius: 0.5em;
        box-shadow: 2px 3px 7px 2px rgba(0,0,0,0.02);
    }
    a:link, a:visited {
        color: #38488f;
        text-decoration: none;
    }
    @media (max-width: 700px) {
        div {
            margin: 0 auto;
            width: auto;
        }
    }
    </style>    
</head>

<body>
<div>
    <h1>Example Domain</h1>
    <p>This domain is for use in illustrative examples in documents. You may use this
    domain in literature without prior coordination or asking for permission.</p>
    <p><a href="https://www.iana.org/domains/example">More information...</a></p>
</div>
</body>
</html>

```

## 参考情報

[Platform/Wio Terminal Network/Wi-Fi - example sketches](https://wiki.seeedstudio.com/Wio-Terminal-Wi-Fi/)  

[Wio Terminalをはじめよう(ピン配置、データシート、回路図など)](https://wiki.seeedstudio.com/jp/Wio-Terminal-Getting-Started/)  

[Platform/Wio Terminal/Network/Overview](https://wiki.seeedstudio.com/Wio-Terminal-Network-Overview/)  
download url:   
https://files.seeedstudio.com/wiki/Wio-Terminal/res/ameba-image-Tool-v2.4.1.zip  
https://files.seeedstudio.com/wiki/Wio-Terminal/res/20200601-rtl8720d-images-v2.2.0.0.zip  

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

以上
