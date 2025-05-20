# HW 7
## RESULT
- TABLE CHART

| Actual Distance(m) | Estimated Distance(m) | ERROR( \|Actual - Estimated\| ) |
| :----------------: | :-------------------: | :-----------------------------: |
|        0.5         |         1.26          |              0.76               |
|        1.0         |         2.00          |              1.00               |
|        2.0         |         0.63          |              1.37               |
|        3.0         |         8.91          |              5.91               |
|        4.0         |         2.24          |              1.76               |

- BAR CHART
![Image](https://github.com/user-attachments/assets/65bd5ac4-1d06-4f49-ad0b-27a4af33956e)

- CLIENT TERMINAL IN IDLE
![Image](https://github.com/user-attachments/assets/06ed855a-473f-43a4-86dd-f0980817097e)

### CODE
- `Server Code`

``` ino
#include <BLEDevice.h>
#include <BLEUtils.h>
#include <BLEServer.h>

void setup() {
  Serial.begin(115200);

  BLEDevice::init("ESP32-Server-team8");   
  BLEServer *pServer = BLEDevice::createServer();

  BLEAdvertising *pAdvertising = pServer->getAdvertising();
  pAdvertising->start();

  Serial.println("BLE Advertising Started...");
}
void loop() {
  delay(1000);
}
```

- `Client Code`

```ino
#include <BLEDevice.h>
#include <BLEScan.h>
#include <BLEAdvertisedDevice.h>
#include <math.h>

BLEScan* pBLEScan;

float calculateDistance(int rssi, int txPower = -59, float n = 2.0) {
  return pow(10.0, ((float)(txPower - rssi)) / (10 * n));
}

class MyAdvertisedDeviceCallbacks: public BLEAdvertisedDeviceCallbacks {
    void onResult(BLEAdvertisedDevice advertisedDevice) {
      String name = advertisedDevice.getName().c_str();
      int rssi = advertisedDevice.getRSSI();

      if (name == "ESP32-Server-team8") { 
        float distance = calculateDistance(rssi);

        Serial.print("Device: ");
        Serial.println(name.c_str());
        Serial.print("RSSI: ");
        Serial.println(rssi);
        Serial.print("Estimated Distance (m): ");
        Serial.println(distance);
        Serial.println("---------------------------");
      }
    }
};

void setup() {
  Serial.begin(115200);
  BLEDevice::init("");
  pBLEScan = BLEDevice::getScan();
  pBLEScan->setAdvertisedDeviceCallbacks(new MyAdvertisedDeviceCallbacks());
  pBLEScan->setActiveScan(true);
  pBLEScan->start(5, false);
}

void loop() {
  pBLEScan->start(5, false);
  delay(5000);
}
```

# ADVANCED 1
## RESULT

![Image](https://github.com/user-attachments/assets/ce5f2a24-08ad-4c7d-9249-90e1ba5839a9)


## CODE   

``` ino
//Client
#include <BLEDevice.h>
#include <BLEScan.h>
#include <BLEAdvertisedDevice.h>
#include <math.h>

BLEScan* pBLEScan;

#define LED_PIN 2  

float calculateDistance(int rssi, int txPower = -59, float n = 2.0) {
  return pow(10.0, ((float)(txPower - rssi)) / (10 * n));
}

class MyAdvertisedDeviceCallbacks: public BLEAdvertisedDeviceCallbacks {
    void onResult(BLEAdvertisedDevice advertisedDevice) {
      std::string name = advertisedDevice.getName();
      int rssi = advertisedDevice.getRSSI();

      if (name == "ESP32-Server-team8") {   
        float distance = calculateDistance(rssi);

        Serial.print("Device: ");
        Serial.println(name.c_str());
        Serial.print("RSSI: ");
        Serial.println(rssi);
        Serial.print("Estimated Distance (m): ");
        Serial.println(distance);
        Serial.println("---------------------------");
      }

      if (distance <= 1.0) {
        digitalWrite(LED_PIN, HIGH); 
        delay(100);
        digitalWrite(LED_PIN, LOW);
        delay(100);
      } else {
        digitalWrite(LED_PIN, LOW); 
      }
    }
};

void setup() {
  Serial.begin(115200);
  pinMode(LED_PIN, OUTPUT);

  BLEDevice::init("");
  pBLEScan = BLEDevice::getScan();
  pBLEScan->setAdvertisedDeviceCallbacks(new MyAdvertisedDeviceCallbacks());
  pBLEScan->setActiveScan(true);
  pBLEScan->start(5, false);
}

void loop() {
  delay(10000);
  pBLEScan->start(5, false);
}


```

# ADVANCED 2
## RESULT
- IN WEB 
![Image](https://github.com/user-attachments/assets/97a0aaa2-c1f8-4c60-96f9-053de6c94e52)

- IN TERMINAL
![Image](https://github.com/user-attachments/assets/5ae23dfb-f12d-45cc-a566-b41537b74fac)


## CODE
```ino
//Client
#include <WiFi.h>
#include <WebServer.h>
#include <BLEDevice.h>
#include <BLEScan.h>
#include <BLEAdvertisedDevice.h>

const char* ssid = "SeungJun"; 
const char* password = "38733873";

BLEScan* pBLEScan;
float currentDistance = 0;

WebServer server(80);

float calculateDistance(int rssi, int txPower = -59, float n = 2.0) {
  return pow(10.0, ((txPower - rssi) / (10 * n)));
}

class MyAdvertisedDeviceCallbacks: public BLEAdvertisedDeviceCallbacks {
    void onResult(BLEAdvertisedDevice advertisedDevice) {
      String name = advertisedDevice.getName().c_str();
      int rssi = advertisedDevice.getRSSI();

      if (name == "ESP32-Server-team8") {
        currentDistance = calculateDistance(rssi);
        Serial.print("Distance: ");
        Serial.println(currentDistance);
      }
    }
};

void handleRoot() {
  String html = "<html><head><meta http-equiv='refresh' content='5'></head><body><h1>Distance to ESP32-Server</h1><p>" +
                String(currentDistance) + " meters</p></body></html>";
  server.send(200, "text/html", html);
}

void setup() {
  Serial.begin(115200);
  BLEDevice::init("");
  pBLEScan = BLEDevice::getScan();
  pBLEScan->setAdvertisedDeviceCallbacks(new MyAdvertisedDeviceCallbacks());
  pBLEScan->setActiveScan(true);
  pBLEScan->start(5, false);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected!");
  Serial.println(WiFi.localIP());

  server.on("/", handleRoot);
  server.begin();
}

void loop() {
  server.handleClient();
  pBLEScan->start(5, false);
  delay(5000);
}

```

