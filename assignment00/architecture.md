# Main technologies of architecture

## Architecture Overview

![IoT Event Streaming Architecture](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*IUaBLlbVKgmsjbjqzew0ZQ.png)

## Eclipse Mosquitto

คือ Open source (EPL/EDL licensed) ที่ใช้สำหรับเป็น Broker ของระบบ MQTT Protocal ซึ่ง Mosquitto นั้นมีขนาดที่เล็ก ซึ่งสามารถใช้งานได้หลากหลายอุปกรณ์ Single Board Computer จนไปถึง เครื่อง Server ซึ่ง MQTT protocol โดยการส่ง Message จะเป็นการใช้ Model publish / subscribe ทำให้เหมาะสำหรับงาน IOT โดยอุปกรณ์ที่เหมาะสำหรับการใช้ MQTT protocol เช่น Sensor ที่ใช้พลังงานน้อย, MCU (Microcontroller) และ มือถือ เป็นต้น

## Apache ZooKeeper

คือ Service ที่เป็นตัวกลางในการจัดการข้อมูล, ชื่อ, Synchronization และ การจัด Group Service โดยจะทำหน้าที่เก็บ Log ทุกอย่างทั้งการ Read / Write โดยรับข้อมูลจาก Kafka

## Apache Kafka

คือ Open-source distributed event-streaming platform ที่ใช้สำหรับการรับส่งข้อมูลแบบ real-time หรือเรียกว่าการทำ “data streaming” ซึ่งเราสามารถใช้ Apache Kafka เป็นเครื่องมือในการทำ process ของ publish (producer ส่งข้อมูลไปที่ topic ที่ต้องการ), store and process (เก็บข้อมูลไว้ที่ Broker (Kafka Cluster)), และ subscribe (consumer รับข้อมูลที่ topic ที่ต้องการ) แบบ real-time ได้
นอกจากนี้ Apache Kafka ยังถูก designed มาให้ใช้งานสำหรับการทำ data streaming ที่ข้อมูลสามารถมาจากหลายแหล่งได้ (หลาย Database) และส่งข้อมูลเป็นชุดใหญ่ ๆ แบบ real-time ได้อีกด้วย ทำให้ข้อดีนี้ เราสามารถนำ Apache Kafka ไปประยุกต์ใช้ในการทำ messaging system สำหรับการรับส่งข้อมูลที่มากได้ในเวลาเดียวกัน

## Apache Kafka rest proxy

คือ Kafka REST Proxy เป็น service ที่ทำหน้าที่เป็น Bridge เชื่อมต่อกับ Kafka cluster และ Client ที่ต้องการเข้าถึง Kafka ผ่าน HTTP (RESTful API) แทนที่จะใช้ Kafka client libraries โดยตรง ซึ่งจะเป็นประโยชน์ในกรณีที่ clients ไม่สามารถเชื่อมต่อกับ Kafka โดยใช้ Kafka protocol ได้ เช่น อยู่ภายนอกเครือข่าย หรือ ถ้าต้องการเข้าใช้งานผ่าน Https

## Apache Kafka Connect

คือ Open-source framework ส่วนหนึ่งที่อยู่ใน Apache Kafka ที่ให้เราสามารถเชื่อมต่อการใช้งาน Kafka กับส่วนอื่น ๆ ของระบบได้ เช่น การทำ streaming data จาก database หรือ data system หนึ่งไปอีกที่หนึ่ง ด้วยการใช้ Kafka เป็นตัวกลางในการรับส่งข้อมูล ซึ่งจะต้องสร้างสิ่งที่เรียกว่า source/sink connector เพื่อให้ Kafka Connect รู้ต้นทางและปลายทางของข้อมูลที่เราอยากจะรับส่ง หรือการนำ search indexes และ file system มาใช้งานกับ Kafka ก็สามารถทำได้เช่นกัน ซึ่งการใช้ Apache Kafka Connect ช่วยให้การทำ streaming data เสถียร, มีประสิทธิภาพสูง, และมีคุณภาพมากขึ้นจากการใช้ connector ที่ต่อกับต้นทางและปลายทางของ data system ที่สามารถเชื่อมต่อตรงไปที่ Kafka ได้เลย

## Apache Kafka Streams (IOT Processor)

คือ library สำหรับการสร้าง application และ microservices ในฝั่ง client ที่ input และ output ของข้อมูลมาจากที่เก็บอยู่ใน cluster ของ Apache Kafka ซึ่งการใช้ Apache Kafka Streams สามารถทำให้เราเขียนและ deploy ตัว applications ในฝั่งของ client และใช้ประโยชน์ของการดึงข้อมูลจาก Kafka Cluster ในฝั่งของ server เพื่อเพิ่มประสิทธิภาพของ application ในการใช้งานได้

## Prometheus

Prometheus อ่านว่า โพรมีธีอุส คือ Software ที่ใช้สำหรับ ติดตาม(monitoring) และ แจ้งเตือน(alerting) เริ่มถูกพัฒนาโดย บริษัท SoundCloud ปัจจุบันถูกเปิดให้เป็น Opensource
มีความสามารถในการ เก็บข้อมูลในลักษณะ multi-dimensional โดยจะจัดเก็บเป็น Time-Series (เรียกตามลำดับเวลา) ค่าที่ไหลเข้ามาจะเป็นในลักษณะ Key/Value โดย Key คือ metric name มี PormQL สำหรับ query และ aggregate ข้อมูล การเก็บข้อมูลใช้ pull model over HTTP (Prometheus เองจะไปดึงข้อมูลจากเป้าหมายที่ต้องการรวบรวมเอง ไม่ต้องให้เป้าหมายส่งมาให้ Prometheus) Prometheus ทำงานได้ดีกับข้อมูล purely numeric time series สามารถ monitor การทำงาน ของเครื่องเป็นหลัก หรือของ server เป็นหลักก็ได้ แต่ละ node ของ Prometheus เป็นแบบ standalone ไม่จำเป็นต้องพึ่งพา network storage หรือ remote service อื่นๆในการทำงาน

## MongoDB

คือ ฐานข้อมูลแบบ No SQL มีคุณสมบัติหลักอย่างหนึ่งของ MongoDB คือ โมเดลข้อมูลเชิงเอกสาร ซึ่งเก็บข้อมูลในรูปแบบของเอกสารคล้าย JSON พร้อม schema เสริม ซึ่งช่วยให้มีความยืดหยุ่นมากขึ้นและใช้เวลาพัฒนาเร็วขึ้น เนื่องจาก schema สามารถแก้ไขได้ง่ายโดยไม่จำเป็นต้องย้ายข้อมูลที่มีราคาแพง นอกจากนั้น MongoDB ยังมีความสามารถในการจัดการข้อมูลจำนวนมาก เนื่องจากสถาปัตยกรรมเป็นแบบกระจายและรองรับการปรับขนาดแนวนอนช่วยให้ปรับขนาดได้อย่างราบรื่นเมื่อขนาดและความซับซ้อนของข้อมูลเพิ่มขึ้น

## Grafana

Grafana อ่านว่า กราฟาน่า คือ software ที่ใช้สำหรับ แสดง ข้อมูล และ จัดรูปแบบ metric data โดยสามารถใช้สร้าง dashboards หรือ charts สำหรับแสดงข้อมูลจากหลาย ๆ แหล่งรวมถึงจาก Time-Series ของ Prometheus ด้วย
กรณีที่ต้องการจะดู request response ของ service ในแต่ละ use case สามารถใช้ Prometheus ในการดึงข้อมูลเข้ามาและใช้ grafana เป็นตัวจัดการสร้าง metrix ขึ้นมาดูเหตุการณ์ และ คาดการณ์เหตุการณ์ที่จะเกิดต่อไปได้เป็นต้น

## IOT Sensor

คือ Service ที่เกิดจากการจำลองค่า Sensors ทั้งอุณหภูมิ ความชื้น แรงดัน และ แสง ที่ทำการสุ่มค่าขึ้นผ่าน Spring Boot แต่ Sensor ค่าต่างๆที่นอกเหนือจากการสุ่มจะมาจาก Board Cucumber โดยส่งค่า Sensor ผ่าน MQTT Protocol
