# Ingest and store real-time data from IoT sensors.

## MQTT Topic


## MQTT Payload


## Set Up Board Cucumber RS
 1.ดาวน์โหลดและติดตั้ง Arduino IDE
 
 2.ติดตั้ง Arduino Core for ESP32 โดยเปิดเมนู File > Preferences แล้วกรอก URL ลงในช่อง Additional Board Manager URLs
 https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
 
 3.ติดตั้ง ESP32 โดยเลือกเมนู Tools > Board > Boards Manager จากนั้นค้น esp32 แล้วติดตั้ง
 
 4.ติดตั้งไดรเวอร์ FTDI แล้วเสียบบอร์ด Cucumber RS กับคอมพิวเตอร์

## ESP32
> 19/06/2567: เขียน Arduino อ่านค่า Sensor จาก Board Cucumber RS (HTS221,BMP280,MPU6050 และ LDR)
25/06/2567: update Readme

```cpp
#include <Wire.h>
#include <Adafruit_BMP280.h>
#include <Adafruit_MPU6050.h>
#include <Adafruit_HTS221.h>
#include <SensirionI2cSht4x.h>

Adafruit_BMP280 bmp;//วัดความดันอากาศแบบสัมบูรณ์ (300 ~ 1100 hPa)
Adafruit_HTS221 hts;//วัดอุณหภูมิ (-40 ~ 120 °C) และความชื้น (0 ~ 100 %RH)
Adafruit_MPU6050 mpu;//วัดตรวจจับการเคลื่อนไหว (ความเร่งแบบ 3 แกน & ไจโรสโคปแบบ 3 แกน)
SensirionI2cSht4x sensor;

//ESP32-S2 ทำงานเป็น I2C master แล้วสั่งอ่าน/เขียนค่ารีจิสเตอร์ของแต่ละเซ็นเซอร์
//ด้วยการระบุค่าแอดเดรส โดยค่าแอดเดรสของ 
//HTS221 คือ 0x5F, BMP280 คือ 0x76 และ MPU-6050 คือ 0x68 

int sensorPin = 5; // select the input pin for the potentiometer
int sensorValue = 0; // variable to store the value coming from the sensor

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
void setup() {
  // put your setup code here, to run once:
  Serial.begin(115200);
  Wire.begin(41,40,100000);
  sensor.begin(Wire,SHT40_I2C_ADDR_44);
  
  setupHardware();
  sensor.softReset();
  delay(10);

  mpu.setAccelerometerRange(MPU6050_RANGE_2_G);
  mpu.setFilterBandwidth(MPU6050_BAND_21_HZ);
  Serial.println("Starting");
}

void loop() {

  //put your main code here, to run repeatedly:
  sensors_event_t temp, humidity, a, g, temp2;
  //hts.getEvent(&humidity, &temp);
  mpu.getEvent(&a, &g, &temp2);

  float aHumidity = 0.0;
  float aTemperature = 0.0;
  sensor.measureHighPrecision(aTemperature,aHumidity);

  Serial.print('\n');
  Serial.print("MPU6050 sensor X: ");
  Serial.print(a.acceleration.x);
  Serial.print(", Y: ");
  Serial.print(a.acceleration.y);
  Serial.print(", Z: ");
  Serial.print(a.acceleration.z);
  Serial.println(" m/s^2");

  Serial.print("HTS221(Sensirion) sensor Temp: ");
  Serial.print(aTemperature);
  Serial.print(" degrees C, Humidity: ");
  Serial.print(aHumidity);
  Serial.println("% rH");
  
  Serial.print("BMP280 sensor Temp: ");
  Serial.print(bmp.readTemperature());
  Serial.print("*C, Pressure: ");
  Serial.print(bmp.readPressure());
  Serial.println("Pa");

  sensorValue = map(analogRead(sensorPin),1900,5000,0,100);
  Serial.print("LDR sensor: ");
  Serial.println(sensorValue, DEC);

  delay(2000);
}

```
