# Ingest and store real-time data from IoT sensors.

## MQTT Topic
Mqtt Topic ผมเซ็ตให้เป็น iot-frames

## MQTT Payload
StaticJsonDocument<512> doc; 
// กำหนดค่าต่าง ๆ ใน JSON
doc["id"] = "43245253";
doc["name"] = "liam-sensor-3";
doc["place_id"] = "42343243";

// รับวันที่และเวลาปัจจุบัน
time_t epochTime = timeClient.getEpochTime();
String formattedDate = formatDate(epochTime);
    
doc["date"] = formattedDate;
doc["timestamp"] = (long long)(epochTime) * 1000 + millis() % 1000;

float p = bmp.readPressure() / 100;
float Temperature = 0.0;
float Humidity = 0.0;
int16_t error = hts.measureLowestPrecision(Temperature, Humidity);
if (error != 0) {
    Serial.println("Erorr func=> measureLowestPrecision()");
}
float t = Temperature;
float h = Humidity;
Luminosity = analogRead(5);
float l = Luminosity * 100 ;

// เพิ่มข้อมูล payload
JsonObject payload = doc.createNestedObject("payload");
payload["temperature"] = t;
payload["humidity"] = h;
payload["pressure"] = p;
payload["luminosity"] = l;

// แปลงเอกสาร JSON เป็นสตริง
char jsonBuffer[512];
serializeJson(doc, jsonBuffer);

## ESP32

```cpp
#include <WiFi.h>
#include <NTPClient.h>
#include <WiFiUdp.h>
#include <ArduinoJson.h>
#include <PubSubClient.h>
#include <SensirionI2cSht4x.h>
#include <Adafruit_BMP280.h>
#include <Adafruit_NeoPixel.h>

Adafruit_BMP280 bmp;
SensirionI2cSht4x hts;

float Luminosity = 0.0;

const char* ssid = "TP-Link_CA0C";            // เปลี่ยนชื่อ Wi-Fi ของคุณ
const char* password = "84722966";    // เปลี่ยนรหัสผ่าน Wi-Fi ของคุณ

// Static IP Configuration
IPAddress local_IP(172, 16, 46, 102);
IPAddress gateway(192, 168, 1, 1);
IPAddress subnet(255, 255, 255, 0);
IPAddress primaryDNS(8, 8, 8, 8);       
IPAddress secondaryDNS(8, 8, 4, 4);  

// MQTT Configuration
const char* mqtt_server = "172.16.46.11";  
const int mqtt_port = 1883;
const char* mqtt_topic = "iot-frames";
const char* mqtt_user = "liam-frames-3"; 
const char* mqtt_password = "1q2w3e4r";

WiFiUDP udp;
NTPClient timeClient(udp, "172.16.46.101", 0 * 3600, 60000); 
WiFiClient espClient;           
PubSubClient client(espClient); 

// กำหนดพอร์ต NeoPixel และจำนวน LED
#define PIN        18    // พอร์ตที่เชื่อมต่อ NeoPixel
#define NUMPIXELS  1     // จำนวนหลอด LED

Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

void setupHardware() {
  Wire.begin(41, 40, 100000);
  if (bmp.begin(0x76)) { // prepare BMP280 sensor
    Serial.println("BMP280 sensor ready");
  }

  hts.begin(Wire, SHT40_I2C_ADDR_44); // prepare HTS221 sensor
  hts.softReset();
  delay(10);
  uint32_t serialNumber = 0;
  int16_t error = hts.serialNumber(serialNumber);    // เรียกใช้เมทอด serialNumber() เพื่ออ่านหมายเลขซีเรียลของเซ็นเซอร์
  if (error != 0) {
    Serial.println("เกิดข้อผิดพลาดในการเรียกใช้ serialNumber()");
  }
  Serial.println("HTS221 sensor ready");

  pixels.begin();  // เริ่มต้นการใช้งาน NeoPixel
  pixels.show();   // ตั้งค่าให้ไฟดับในตอนเริ่มต้น
}

// ฟังก์ชันเปลี่ยนสี LED ของ NeoPixel
void setPixelColor(uint32_t color) {
  pixels.setPixelColor(0, color);
  pixels.show();
}

// ฟังก์ชันเพื่อฟอร์แมตวันที่ในรูปแบบ ISO 8601
String formatDate(time_t epochTime) {
  struct tm *ptm = gmtime(&epochTime);
  int milliseconds = millis() % 1000;

  char buffer[30];
  sprintf(buffer, "%04d-%02d-%02dT%02d:%02d:%02d.%03d+0000",
          ptm->tm_year + 1900, 
          ptm->tm_mon + 1,     
          ptm->tm_mday,        
          ptm->tm_hour,        
          ptm->tm_min,         
          ptm->tm_sec,         
          milliseconds);       

  return String(buffer);
}

void setup() {
  Serial.begin(115200);
  setPixelColor(pixels.Color(0, 255, 0));
  setupHardware();
  // เชื่อมต่อ Wi-Fi
  if (!WiFi.config(local_IP, gateway, subnet, primaryDNS, secondaryDNS)) {
    Serial.println("STA Failed to configure");
  }

  WiFi.begin(ssid, password);
  
  timeClient.begin();
  while(!timeClient.update()) {
    Serial.println("Waiting for NTP time sync...");
    delay(1000);  
  }  
}

void loop() {
  setPixelColor(pixels.Color(0, 255, 0)); // แดงสำหรับ WiFi เชื่อมต่อ
  // ลูปนอกสุด: เช็คว่าเชื่อมต่อ Wi-Fi แล้วหรือยัง
  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("WiFi connected");
    Serial.println("IP address: ");
    Serial.println(WiFi.localIP());


    // ลูปชั้นกลาง: เช็คว่าเชื่อมต่อ MQTT แล้วหรือยัง
    setPixelColor(pixels.Color(0, 0, 255));  // สีฟ้าสำหรับ MQTT เชื่อมต่อ
    client.setServer(mqtt_server, mqtt_port);
    if (!client.connected()) {
      if (client.connect("ESP32Client", mqtt_user, mqtt_password)) {
        Serial.println("MQTT connected"); 
      } else {
        setPixelColor(pixels.Color(0, 0, 255));  // สีฟ้าสำหรับ MQTT เชื่อมต่อ
        Serial.print("MQTT connection failed");
        Serial.println(client.state());
        delay(5000);  // รอ 5 วินาทีแล้วลองใหม่
      }
    }

    // ลูปชั้นในสุด: กำลังส่งข้อมูลอยู่ใช่ไหม
    bool isSendingData = true;  // คุณสามารถเปลี่ยนเงื่อนไขนี้ได้ตามที่ต้องการ
    if (isSendingData) {
      // สร้างวัตถุ JSON
      StaticJsonDocument<512> doc; 

      // กำหนดค่าต่าง ๆ ใน JSON
      doc["id"] = "43245253";
      doc["name"] = "liam-sensor-3";
      doc["place_id"] = "42343243";

      // รับวันที่และเวลาปัจจุบัน
      time_t epochTime = timeClient.getEpochTime();
      String formattedDate = formatDate(epochTime);
    
      doc["date"] = formattedDate;
      doc["timestamp"] = (long long)(epochTime) * 1000 + millis() % 1000;

      float p = bmp.readPressure() / 100;
      float Temperature = 0.0;
      float Humidity = 0.0;
      int16_t error = hts.measureLowestPrecision(Temperature, Humidity);
      if (error != 0) {
        Serial.println("Erorr func=> measureLowestPrecision()");
      }
      float t = Temperature;
      float h = Humidity;
      Luminosity = analogRead(5);
      float l = Luminosity * 100 ;

      // เพิ่มข้อมูล payload
      JsonObject payload = doc.createNestedObject("payload");
      payload["temperature"] = t;
      payload["humidity"] = h;
      payload["pressure"] = p;
      payload["luminosity"] = l;

      // แปลงเอกสาร JSON เป็นสตริง
      char jsonBuffer[512];
      serializeJson(doc, jsonBuffer);

      // ส่งข้อมูล JSON ไปยัง MQTT topic
      if (client.publish(mqtt_topic, jsonBuffer)) {
        // แสดงผล JSON
        Serial.println(jsonBuffer);
        Serial.println("Data sent successfully.");
        setPixelColor(pixels.Color(255, 0, 0));  // เขียวสำหรับการส่งข้อมูล
      } else {
        Serial.println("Data send failed.");
      }
    } else {
      Serial.println("Not sending data");
    }
  } else {
    setPixelColor(pixels.Color(0, 255, 0)); // แดงสำหรับ WiFi เชื่อมต่อ
    Serial.println("WiFi not connected");
    // ลองเชื่อมต่อ Wi-Fi ใหม่
    WiFi.begin(ssid, password);
    delay(5000);  // รอ 5 วินาทีแล้วลองใหม่
  }

  delay(1000);  // รอ 1 วินาทีก่อนเริ่มลูปใหม่
}

```