# Ingest and store real-time data from IoT sensors.

## MQTT Topic

> MQTT Topic = "iot-frames"

## MQTT Payload

MQTT Payload:

```cpp
  StaticJsonDocument<200> doc;
  doc["id"] = "43245253";
  doc["name"] = "liam-sensor-3";
  doc["place_id"] = "42343243";
  doc["date"] = NTP.getTimeDateString(time(NULL),"%Y-%m-%dT%H:%M:%S");
  doc["timestamp"] = epochTime;

  JsonObject payload = doc.createNestedObject("payload");
  payload["temperature"] = aTemperature;
  payload["humidity"] = aHumidity;
  payload["pressure"] = bmpPressure / 100;
  payload["luminosity"] = sensorValue;
```

## Set Up Board Cucumber RS
> 1. ดาวน์โหลดและติดตั้ง Arduino IDE
2. ติดตั้ง Arduino Core for ESP32 โดยเปิดเมนู File > Preferences แล้วกรอก URL ลงในช่อง Additional Board Manager URLs
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
3. ติดตั้ง ESP32 โดยเลือกเมนู Tools > Board > Boards Manager จากนั้นค้น esp32 แล้วติดตั้ง
4. ติดตั้งไดรเวอร์ FTDI แล้วเสียบบอร์ด Cucumber RS กับคอมพิวเตอร์

## Send data to kafka
>1. include library สำหรับเชื่อมต่อกับ MQTT broker และ ส่งข้อมูลเซ็นเซอร์ในรูปแบบ JSON (PubSubClient.h, ArduinoJson.h, WiFi.h)
2. เชื่อมต่อกับ WiFi ของเราโดยสร้าง function setupWifi()
3. เชื่อมต่อกับ MQTT broker โดยผ่าน function reconnect()
4. สร้าง JSON จากข้อมูลเซ็นเซอร์ HTS221, BMP280, MPU6050
5. ส่ง JSON ไปยัง topic ของเรา

## ESP32
> 19/06/2567: เขียน Arduino อ่านค่า Sensor จาก Board Cucumber RS (HTS221,BMP280,MPU6050 และ LDR)

>25/06/2567: update Readme

>25/08/2567: update Readme

```cpp
// send ค่าผ่าน MQTT 
#include <Wire.h>
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>

// read ค่าจาก senser
#include <Adafruit_BMP280.h>
#include <Adafruit_MPU6050.h>
#include <Adafruit_HTS221.h>
#include <SensirionI2cSht4x.h>

// ตั้งค่าสี
#include <Adafruit_NeoPixel.h> 

// ตั้งค่า time
#include <time.h>
#include <ESPNtpClient.h>
// #include <NTPClient.h>
// #include <WiFiUdp.h>

#define NTP_TIMEOUT 5000

#define NUM_PIXELS  (8)            // The number of pixels
#define WS2812_PIN  GPIO_NUM_18 // The ESP32 pin for DATA output

// Create an object from the 'Adafruit_NeoPixel' class 
Adafruit_NeoPixel pixels(NUM_PIXELS, WS2812_PIN, NEO_RGB + NEO_KHZ800);

//ตั้งค่าตัวแปร senser
Adafruit_BMP280 bmp;//วัดความดันอากาศแบบสัมบูรณ์ (300 ~ 1100 hPa)
Adafruit_HTS221 hts;//วัดอุณหภูมิ (-40 ~ 120 °C) และความชื้น (0 ~ 100 %RH)
Adafruit_MPU6050 mpu;//วัดตรวจจับการเคลื่อนไหว (ความเร่งแบบ 3 แกน & ไจโรสโคปแบบ 3 แกน)
SensirionI2cSht4x sensor;

const PROGMEM char* ntpServer = "172.16.46.11"; //ใช้ ntp server จาก server ของเรา 

int sensorPin = 5; // select the input pin for the potentiometer
int sensorValue = 0; // variable to store the value coming from the sensor

// Colors for different states
const uint32_t LED_COLOR_STARTING = 0x00FF00; // Green
const uint32_t LED_COLOR_WIFI_CONNECTED = 0x0000FF; // Blue
const uint32_t LED_COLOR_MQTT_CONNECTED = 0xFFFF00; // Yellow
const uint32_t LED_COLOR_SENDING_DATA = 0x8F8F8F; // While
const uint32_t LED_COLOR_ERROR = 0xFF0000;// Red

//WIFI
const char* ssid = "TP-Link_7CC6";//TP-Link_7CC6
const char* password = "08378774";//08378774 

//set Mqtt server
const char* mqtt_server = "172.16.46.11";
const int mqttPort = 1883;
const char* mqttUser = "liam-sensor-3";
const char* mqttPassword = "1q2w3e4r";

//set up ip address
IPAddress ip(172, 16, 46, 17);  
IPAddress gateway(172,16,46,254);
IPAddress subnet(255, 255, 255, 0);


WiFiClient espClient;
PubSubClient client(espClient);

void setupHardware(){
  //Wire.begin(41,40,100000);
  if (bmp.begin(0x76)){
    Serial.println("BMP280 sensor ready");
  }
  
  if (hts.begin_I2C()){
    Serial.println("HTS221 sensor ready");
    //while(1);
  }
  if (mpu.begin()) { // prepare MPU6050 sensor
   Serial.println("MPU6050 sensor ready!");
  }
  pinMode(2,OUTPUT);
  digitalWrite(2,HIGH);
}

void setLED(uint32_t color) {
  for (uint16_t i = 0; i < NUM_PIXELS; i++) {
    pixels.setPixelColor(i, color);
  }
  pixels.show();
}

void blinkLED(uint32_t color, int blinkTimes, int delay_us) {
  for (int i = 0; i < blinkTimes; i++) {
    setLED(color);           // Turn on the LED with the specified color
    delayMicroseconds(delay_us); // Wait for the specified microseconds
    setLED(0x000000);        // Turn off the LED
    delayMicroseconds(delay_us); // Wait again
  }
}

void setupWifi() {
  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  WiFi.config(ip, gateway, subnet);


  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
    blinkLED(LED_COLOR_ERROR, 10, 50000);// Indicate error while connecting to WiFi
  }

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  setLED(LED_COLOR_WIFI_CONNECTED); // Indicate successful WiFi connection
}
void reconnect(){
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.println("Connecting to MQTT...");
 
    if (client.connect("idea_senser3_is_here",mqttUser,mqttPassword)) {
      
      Serial.println("connected");
      setLED(LED_COLOR_MQTT_CONNECTED); // Indicate successful MQTT connection

    } else {
      setLED(LED_COLOR_ERROR); // Indicate error when failing to connect to MQTT
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void callback(char* topic, byte* message, unsigned int length) {
  Serial.print("Message arrived on topic: ");
  Serial.print(topic);
  Serial.print(". Message: ");
  String messageTemp;
  
  for (int i = 0; i < length; i++) {
    Serial.print((char)message[i]);
    messageTemp += (char)message[i];
  }
  Serial.println();
}

unsigned long Get_Epoch_time(){
  time_t now;
  struct tm timeinfo;
  if(!getLocalTime(&timeinfo)){
    return 0;
  }
  time(&now);
  return now;
}


void setup() {
  // put your setup code here, to run once:
  Serial.begin(115200);
  Wire.begin(41,40,100000);
  sensor.begin(Wire,SHT40_I2C_ADDR_44);

  pixels.begin(); // Initialize the NeoPixel WS2812 strip.
  pixels.setBrightness(255); // Set the brightness to 255.

  setLED(LED_COLOR_STARTING);
  
  setupHardware();
  sensor.softReset();
  delay(10);

  mpu.setAccelerometerRange(MPU6050_RANGE_2_G);
  mpu.setFilterBandwidth(MPU6050_BAND_21_HZ);
  Serial.println("Starting");

  setupWifi();
  
  client.setServer(mqtt_server, mqttPort);
  client.setCallback(callback);

  
  // Init and get the time
  // configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);
  NTP.setTimeZone (TZ_Asia_Bangkok);
  NTP.setInterval(600);
  NTP.setNTPTimeout(NTP_TIMEOUT);
  NTP.begin (ntpServer);

}

void loop() {
  // put your main code here, to run repeatedly:
  if (WiFi.status() != WL_CONNECTED){
    setupWifi();
  }
  if (!client.connected()) {
      reconnect();
  }
  client.loop();

//========================= Get Data from Sensers ========================
  sensors_event_t temp, humidity, a, g, temp2;
  mpu.getEvent(&a, &g, &temp2);

  //get HTS221(Sensirion)(วันอุณหภูมิ,ความชื่น)
  float aHumidity = 0.0;
  float aTemperature = 0.0;
  sensor.measureHighPrecision(aTemperature, aHumidity);

  //get bmp(วัดความดันอากาศ)
  float bmpTemperature = bmp.readTemperature();
  float bmpPressure = bmp.readPressure();

  //get mpu(วัดตรวจจับการเคลื่อนไหว)
  float X_value = a.acceleration.x;
  float Y_value = a.acceleration.y;
  float Z_value = a.acceleration.z;

  //get LDR (วัดแสง)
  //sensorValue = map(analogRead(sensorPin),1600,6000,0,65);
  sensorValue = analogRead(sensorPin);
//======================== สร้าง JSON ====================================
  //StaticJsonDocument<200> doc;
  // doc["mpu"]["x"] = X_value;
  // doc["mpu"]["y"] = Y_value;
  // doc["mpu"]["z"] = Z_value;
  // doc["bmp280"]["temperature"] = bmpTemperature;
  // doc["bmp280"]["pressure"] = bmpPressure;
  // doc["hts221(sht4x)"]["temperature"] = aTemperature;
  // doc["hts221(sht4x)"]["humidity"] = aHumidity;
  // doc["ldr"] = sensorValue;
  unsigned long epochTime = Get_Epoch_time();

  StaticJsonDocument<200> doc;
  doc["id"] = "90826450";//43245253
  doc["name"] = "iot_sensor_9";//liam-sensor-3
  doc["place_id"] = "32347983";//42343243
  doc["date"] = NTP.getTimeDateString(time(NULL),"%Y-%m-%dT%H:%M:%S");
  doc["timestamp"] = epochTime;

  JsonObject payload = doc.createNestedObject("payload");
  payload["temperature"] = aTemperature;
  payload["humidity"] = aHumidity;
  payload["pressure"] = bmpPressure / 100;
  payload["luminosity"] = sensorValue;

  char buffer[256];
  serializeJson(doc, buffer);

  // Simulate data sending process
  blinkLED(LED_COLOR_SENDING_DATA, 10, 50000); // Blink while 10 times with 50ms on/off

  // ส่ง JSON ไปยัง topic ที่กำหนด
  // client.publish("iot-frames", buffer);
  if (!client.publish("iot-frames", buffer)) {
    Serial.println("Failed to send message");
    setLED(LED_COLOR_ERROR); // Indicate error if message fails to send
  } else {
    Serial.println("Message sent successfully");
  }
 
  Serial.print("date: ");
  Serial.print(NTP.getTimeDateString(time(NULL),"%Y-%m-%dT%H:%M:%S"));

  Serial.print("time stamp: ");
  Serial.print(epochTime);

  Serial.print("HTS221(Sensirion) sensor Temp: ");
  Serial.print(aTemperature);
  Serial.print(" degrees C, Humidity: ");
  Serial.print(aHumidity);
  Serial.println("% rH");

  Serial.print("BMP280 sensor Temp: ");
  Serial.print(bmp.readTemperature());
  Serial.print("*C, Pressure: ");
  Serial.print(bmpPressure/100);
  Serial.println("Pa");

  Serial.print("LDR sensor: ");
  Serial.println(sensorValue, DEC);


  delay(2000);

}


```
