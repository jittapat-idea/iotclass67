# MQTT

## iot-sensor-1

IoT Sensor 1 เป็น IoT ที่ได้มาจากการ compose up docker ใน Server ตัวมันเอง ซึ่งมาจาก image IOT senser ซึ่งจะถูกสุ่มค่าด้วย Spring Boot

## iot-sensor-2

IoT Sensor 2 เป็น IoT ที่ได้มาจากการ compose up docker ใน PC ตัวนอกที่ได้ทำการส่งข้อมูลไปยัง Server ใช้ image IOT senser และสุ่มค่าด้วย Spring Boot เช่นเดียวกับ iot-sensor-1

## iot-sensor-3-10

IoT Sensor 3 ถึง 10 เป็น IoT ที่ได้ข้อมูลจาก sensor จริงๆ ผ่าน board cumcuber โดยตัวที่ 3 จะเป็นค่าที่ได้จาก board ของกลุ่มเราเอง ตัวที่ 4 - 10 จะเป็นค่าที่ได้จาก board ของกลุ่มอื่นๆ ที่ส่งมาให้เราอีก
