
2020/8/9:   
ChestTypeとWristTypeの両方対応にスケッチを改善した。

2020/8/8+  

PlatformIO M5Atom BLE HR Sensor Receiver
# PlatformIO M5Atom BLE HR Sensor Receiver

## 概要
M5Atom(ESP32を含む)でHRセンサーデータを受信する(Arduino版)    
開発ツールのインストールについては「[M5Atomを開発ツールPlatformIOで使う(M5Atom/Arduino版)](https://beta-notes.way-nifty.com/blog/2020/08/post-3486f7.html)」を参照のこと。
(ホストPCとしてはubuntuを想定している)


## platformio.ini
platformio.iniを以下のように編集する：
```

; PlatformIO Project Configuration File
;
;   Build options: build flags, source filter
;   Upload options: custom upload port, speed and extra flags
;   Library options: dependencies, extra library storages
;   Advanced options: extra scripting
;
; Please visit documentation for the other options and examples
; https://docs.platformio.org/page/projectconf.html

[env:esp32dev]
platform = espressif32
board = m5stick-c
framework = arduino

monitor_speed = 115200

lib_deps =
    # use M5Atom lib
    3113
    # use "FastLED"
    126

lib_ldf_mode = deep+

```
M5Atom特有のライブラリを使用していないので、ESP32で動作させる場合は
通常のESP32のplatformio.iniで良い。

## テスト用スケッチをダウンロードする

```

cd src
wget https://cdn.hackaday.io/files/880553767345120/Pulse.ino
```

## テスト・スケッチ
ダウンロードしたスケッチを以下の修正点に対応して変更する：   

修正点:   
(1)余計なスペースを削除する  
(2)Wrist-TypeのHRセンサー対応を追加する  
(3)HRデータを表示する際、タイムスタンプ表示を追加する  
(4)LED点滅をコメントアウトする

src/Pluse.ino(編集済み)   
\#余計なスペースを削除するなど大幅な修正になっているので   
\#オリジナルを修正するよりも、以下の修正済みのものをコピー＆ペーストすることを勧める：
```c++


// modified/bufix 2020/8/9 by: xshige
#define CHEST
//#define WRIST

// fix info:
//   https://github.com/nkolban/esp32-snippets/issues/837
//   Using BLE for Heart Rate Monitors, newer belts don't work #837

unsigned long startMillis;

/************************************************************************/
/****************** Heart Rate BLE gateway for Technogym ****************/
/************ Outputs a 30 ms pulse at recieved HR frequency ************/
/**************** Indicates the remote HR battery level *****************/
/*********************** Vincent HORN - march 2018 **********************/
/**************************** version 1.0 *******************************/

#include "BLEDevice.h"

// Device connection
static BLEAddress *pServerAddress;
BLEClient*  pClient;
static boolean doConnect = false;
static boolean isconnect = false;

// The remote HR service & characteristic
static BLEUUID service_HR_UUID(BLEUUID((uint16_t)0x180D));
static BLEUUID char_HR_UUID(BLEUUID((uint16_t)0x2A37));
static BLERemoteCharacteristic* pRemote_HR_Characteristic;
const uint8_t notificationOn[] = {0x1, 0x0};

// HR Pulse timer
hw_timer_t * timer = NULL; // Timer 
volatile SemaphoreHandle_t timerSemaphore; // Timer 
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;// Timer 
volatile uint32_t isrCounter = 0; // Timer 
volatile uint32_t lastIsrAt = 0; // Timer 

// HR Pulse output
int ms_pb = 500; //millis per beat
const int pulse_pin = 23; // pulse output pin
bool pulse_on = false; // flag when pulse have to be fired
unsigned long pulse_off_at = 0; // Time to stop the pulse

// The remote BATT service a characteristic
static BLEUUID service_BATT_UUID(BLEUUID((uint16_t)0x180F));
static BLEUUID char_BATT_UUID(BLEUUID((uint16_t)0x2A19));
static BLERemoteCharacteristic* pRemote_BATT_Characteristic;

// BATT indicator
const int battery_scan = 60000; // Scan interval in msec.
unsigned long batt_read_start_at = 0;
bool batt_indicator = false; // time to indicate de battery level
int batt_flash = 0; // number of flash
int i_flash = 1; // flash counter
bool init_time = true; // flag for init on/off time  
unsigned long on_at = 0;	// Time to turn LED ON
unsigned long off_at = 0; // Time to turn LED OFF
const int led_pin = 22; // Battery indicator LED

static void notifyCallback(BLERemoteCharacteristic* pBLERemoteCharacteristic, uint8_t* pData, size_t length, bool isNotify) {

	uint8_t HR_val = 0; // RAW HR value
	
        //< support both Chest/Wrist type 2020/8/9 by: xshige
	//if (pData[0] == 22 || pData[0] == 6) {
	if (pData[0] == 16) { // for Chest Type HR sensor (Polar H10)	
		if (pData[1] > 10 && pData[1] < 254) {// HR must be between 10 and 254 bpm
			HR_val = pData[1];
		} else HR_val =0;
	} else if (pData[0] == 0) { // for Wrist Type HR sensor	(Polar A370, OH1)
		if (pData[1] > 10 && pData[1] < 254) {// HR must be between 10 and 254 bpm
			HR_val = pData[1];
		} else HR_val =0;
	} else HR_val = 0;
        //>

// display timestamp(sec)
unsigned long currentMillis = millis()-startMillis;
//Serial.println(pData[0]);
Serial.print(currentMillis/1000);
Serial.print(": ");
//
	Serial.print("HR : ");
	Serial.print(HR_val);
	Serial.print(" bpm \t Battery : ");
	Serial.print(batt_flash*10);
	Serial.println(" %");
	
	if (HR_val != 0) {ms_pb = (int)60000/HR_val;} 
        // interval in ms between 2 heart beat	if no value (zero) is recieved, ms_pb is set to 100s

}

bool connectToServer(BLEAddress pAddress) {
	
	Serial.print("Forming a connection to ");
	Serial.println(pAddress.toString().c_str());
	
	// create a new client
	pClient  = BLEDevice::createClient();
	Serial.println("Created client");

	// Connect to the remove BLE Server.
#ifdef CHEST
        pClient->connect(pAddress, (esp_ble_addr_type_t)1); // bug fix 2020/8/9 (Chest type can be OK)
#endif
#ifdef WRIST
	pClient->connect(pAddress); // Chest can Not be OK
#endif

	Serial.println("Connected to server");

	// Obtain a reference to the HR service of the remote BLE server.
	BLERemoteService* pRemote_HR_Service = pClient->getService(service_HR_UUID);
	if (pRemote_HR_Service == nullptr) {
		Serial.print("Failed to find HR service : ");
		//Serial.println(service_HR_UUID.toString().c_str());
		return false;
	} else Serial.println("Found HR service");

	// Obtain a reference to the BATT service of the remote BLE server.
	BLERemoteService* pRemote_BATT_Service = pClient->getService(service_BATT_UUID);
	if (pRemote_BATT_Service == nullptr) {
		Serial.print("Failed to find BATT service : ");
		//Serial.println(service_BATT_UUID.toString().c_str());
		return false;
	} else Serial.println("Found BATT service");

	// Obtain a reference to the HR characteristic in the service of the remote BLE server.
	pRemote_HR_Characteristic = pRemote_HR_Service->getCharacteristic(char_HR_UUID);
	if (pRemote_HR_Characteristic == nullptr) {
		Serial.print("Failed to find HR characteristic : ");
		//Serial.println(char_HR_UUID.toString().c_str());
		return false;
	} else Serial.println("Found HR characteristic");

	// Obtain a reference to the BATT characteristic in the service of the remote BLE server.
	pRemote_BATT_Characteristic = pRemote_BATT_Service->getCharacteristic(char_BATT_UUID);
	if (pRemote_BATT_Characteristic == nullptr) {
		Serial.print("Failed to find BATT characteristic : ");
		//Serial.println(char_BATT_UUID.toString().c_str());
		return false;
	} else Serial.println("Found BATT characteristic");

	// Set notification
	pRemote_HR_Characteristic->registerForNotify(notifyCallback);
	pRemote_HR_Characteristic->getDescriptor(BLEUUID((uint16_t)0x2902))->writeValue((uint8_t*)notificationOn, 2, true);

}

class MyAdvertisedDeviceCallbacks: public BLEAdvertisedDeviceCallbacks {
	
	void onResult(BLEAdvertisedDevice advertisedDevice) {
		Serial.print("BLE Advertised Device found: ");
		Serial.println(advertisedDevice.toString().c_str());

		// We have found a device, let us now see if it contains the service we are looking for.
		if (advertisedDevice.haveServiceUUID() && advertisedDevice.getServiceUUID().equals(service_HR_UUID)) {

			Serial.println("Found Heart Rate device! ");
			advertisedDevice.getScan()->stop();

			pServerAddress = new BLEAddress(advertisedDevice.getAddress());
			doConnect = true;
		}	
	}
};

void device_scan(){
	
	// Retrieve a Scanner and set the callback we want to use to be informed when we
	// have detected a new device.  Specify that we want active scanning and start the
	// scan to run for 15 seconds.
	
	BLEScan* pBLEScan = BLEDevice::getScan();
	pBLEScan->setAdvertisedDeviceCallbacks(new MyAdvertisedDeviceCallbacks());
	pBLEScan->setActiveScan(true);
	
	while (!doConnect) {
		// Scan till a connection is found
		Serial.println("Scanning for BLE devices ...");
		BLEScanResults foundDevices = pBLEScan->start(15);
		Serial.print("Devices found: ");
		Serial.println(foundDevices.getCount());
	}
}

void IRAM_ATTR onTimer(){
	
	// Increment the counter and set the time of ISR
	portENTER_CRITICAL_ISR(&timerMux);
	isrCounter++;
	lastIsrAt = millis();
	portEXIT_CRITICAL_ISR(&timerMux);
	
	// Give a semaphore that we can check in the loop
	xSemaphoreGiveFromISR(timerSemaphore, NULL);
	// It is safe to use digitalRead/Write here if you want to toggle an output
}

void setup() {
	
	Serial.begin(115200);
	Serial.println("------------------------------------------");
	Serial.println("Starting Arduino BLE Client application...");

	// initialize pulse_pin as an output
	//pinMode(pulse_pin, OUTPUT);
	
	// initialize battery pin led as an output
	//pinMode(led_pin, OUTPUT);
	
	// initialize timer 
	timerSemaphore = xSemaphoreCreateBinary(); // Create semaphore to inform us when the timer has fired
	timer = timerBegin(0, 80, true);
	
	// Attach onTimer function to our timer.
	timerAttachInterrupt(timer, &onTimer, true);
  
	// initialize BLE 
	BLEDevice::init("");

	// Scan for devices
	device_scan();

startMillis = millis();

}

void loop() {

	// If the flag "doConnect" is true then we have scanned for and found the desired BLE Server with which we wish to connect.  
	if (doConnect == true) {
		if (connectToServer(*pServerAddress)) {
			
			Serial.println("Connected to the BLE Server.");
			isconnect = true;
			
			// First timer set to 10 sec.
			timerAlarmWrite(timer, 10000000, true);
			timerAlarmEnable(timer);
			
		} else {
			Serial.println("We have failed to connect to the server; there is nothin more we will do.");
			isconnect = false;
		}
		doConnect = false;
	}

	// when interrupt occurres
	if (xSemaphoreTake(timerSemaphore, 0) == pdTRUE){
		uint32_t isrCount = 0, isrTime = 0;
		
		// Read the interrupt count and time
		portENTER_CRITICAL(&timerMux);
		isrTime = lastIsrAt;
		isrCount = isrCounter;
		portEXIT_CRITICAL(&timerMux);
		
		// create a 30 ms pulse on pulse_pin 
		pulse_on = true;
		pulse_off_at = millis() + 30;
		
		//digitalWrite(pulse_pin, HIGH);
		//delay(30);
		//digitalWrite(pulse_pin, LOW);
		
		// rearm the timer with previous incomming ms per beat (ms_pb) value
		timerAlarmDisable(timer);
		timerAlarmWrite(timer, 1000*ms_pb, true);
		timerAlarmEnable(timer);
		
		/*
		Serial.print("at ");
		Serial.print(isrTime);
		Serial.print(" ms");
		Serial.print("\t new timer ");
		Serial.print(ms_pb);
		Serial.println(" ms");
		*/
	}
	
	// Create Pulse 
	if (pulse_on) {
		if (millis() < pulse_off_at) {
			//digitalWrite(PULSE_PIN, HIGH);   // turn the LED on (HIGH is the voltage level)
		}
		else {
			//digitalWrite(PULSE_PIN, LOW);    // turn the LED off by making the voltage LOW
			pulse_on = false;
		}
	}
		
	
	// Read the value of the battery level characteristic each battery_scan ms (only if connected)
	if (isconnect && (millis()-batt_read_start_at > battery_scan)) {
		std::string value = pRemote_BATT_Characteristic->readValue();
		String batt_caracteristic = value.c_str();
		
		// convert string to char
		char buf[2];
		batt_caracteristic.toCharArray(buf, 2);
		
		// BATT_val is the numeric ASCII value
		int BATT_val = int(buf[0]);
		
		// batt_flash number of flash (30% battery => 3 flashs)
		batt_flash = (int)BATT_val/10;
		
		// fire the LED
		batt_indicator = true;
	
		batt_read_start_at = millis();
	}

	// Battery indicator
	if (batt_indicator){
		
		if (i_flash <= batt_flash){
					
			if (init_time){
				off_at = millis() + 40;
				on_at = off_at + 500;
				init_time = false;
			}
			
			if (millis() <= off_at){
				digitalWrite(led_pin, HIGH);
			}
			
			if ((millis() > off_at) && (millis() <= on_at)){
				digitalWrite(led_pin, LOW);
			} else if (millis() > on_at){
				init_time = true;
				i_flash++;
			}

		} else {
			i_flash = 1;
			batt_indicator = false;
		}
	}
	
	// Watch if client is still connected. When not reboot ESP32
	if (pClient->isConnected() == false) {
		Serial.println("Device disconnected ! ESP 32 rebooting .....");
		ESP.restart();
	}
}
```
使用するHRセンサーのタイプで以下の#defineのうち一つを有効にして、コンパイル＆実行する。
```

#define CHEST
#define WRIST
```
書き込み後、実行するとBLEデバイスをスキャンしてHRセンサーが見つかれば自動的にペアリングして、HRデータを表示する。
既にHRセンサーが（スマフォなどと）ペアリング済みだと接続できないので、その場合、該当のスマフォのペアリングを解除する。

出力例：(Polar A370の場合)
```
picocom /dev/ttyUSB0 -b115200

Starting Arduino BLE Client application...
Scanning for BLE devices ...
BLE Advertised Device found: Name: Polar A370 1D44A125, Address: a0:9e:1a:1d:44:a1, manufacturer data: 6b002f0000, serviceUUID: 0000180d-0000-1000-8000-00805f9b34fb
Found Heart Rate device! 
Devices found: 1
Forming a connection to a0:9e:1a:1d:44:a1
Created client
Connected to server
Found HR service
Found BATT service
Found HR characteristic
Found BATT characteristic
Connected to the BLE Server.
1: HR : 69 bpm 	 Battery : 0 %
2: HR : 69 bpm 	 Battery : 0 %
3: HR : 70 bpm 	 Battery : 0 %
4: HR : 70 bpm 	 Battery : 0 %
5: HR : 69 bpm 	 Battery : 0 %
6: HR : 69 bpm 	 Battery : 0 %
7: HR : 70 bpm 	 Battery : 0 %
8: HR : 70 bpm 	 Battery : 0 %
9: HR : 70 bpm 	 Battery : 0 %
# バッテリ残量が0%なのは、バッテリ残量を取得するのに時間がかかるせいのようだ。
# 待っていると自動的に表示されるようになる。
```

出力例：(Polar H10の場合)
```

picocom /dev/ttyUSB0 -b115200

Starting Arduino BLE Client application...
Scanning for BLE devices ...
BLE Advertised Device found: Name: , Address: 7d:01:bf:c3:6f:ff, manufacturer data: 4c0010050318e24cea
BLE Advertised Device found: Name: Polar H10 31BFD72E, Address: fd:e0:ff:99:cc:1e, manufacturer data: 6b00330c4243
Devices found: 2
Scanning for BLE devices ...
BLE Advertised Device found: Name: , Address: fd:e0:ff:99:cc:1e, manufacturer data: 6b002f0c4343, serviceUUID: 0000180d-0000-1000-8000-00805f9b34fb, txPower: 4
Found Heart Rate device! 
Devices found: 1
Forming a connection to fd:e0:ff:99:cc:1e
Created client
Connected to server
Found HR service
Found BATT service
[E][BLERemoteCharacteristic.cpp:274] retrieveDescriptors(): esp_ble_gattc_get_all_descr: Unknown
Found HR characteristic
Found BATT characteristic
Connected to the BLE Server.
2: HR : 67 bpm 	 Battery : 0 %
3: HR : 66 bpm 	 Battery : 0 %
4: HR : 66 bpm 	 Battery : 0 %
5: HR : 66 bpm 	 Battery : 0 %
6: HR : 66 bpm 	 Battery : 0 %
7: HR : 66 bpm 	 Battery : 0 %
8: HR : 66 bpm 	 Battery : 0 %
9: HR : 67 bpm 	 Battery : 0 %
# バッテリ残量が0%なのは、バッテリ残量を取得するのに時間がかかるせいのようだ。
# 待っていると自動的に表示されるようになる。
```

## 参考情報

https://hackaday.io/project/88055-technogym-new-heart-rate-ble-sensor  
Technogym : new Heart Rate BLE sensor   
Replace the RF HR receiver with a new BLE sensor based on ESP32.

[Web-BluetoothでHRセンサーを接続する](https://beta-notes.way-nifty.com/blog/2019/11/post-a93ab9.html)  
[Web-Bluetooth:HR(心拍)センサーのロガー](https://beta-notes.way-nifty.com/blog/2019/11/post-b631c9.html)   
[Web-Bluetooth:CSC(自転車)センサーのロガー](https://beta-notes.way-nifty.com/blog/cat24243606/index.html)   

[PlatformIO Core (CLI)](https://docs.platformio.org/en/latest/core/index.html)  

以上
