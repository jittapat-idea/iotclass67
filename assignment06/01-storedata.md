# Store data.

การเก็บข้อมูลลงในฐานข้อมูลนั้น จะทำการดึงข้อมูลจาก Kafka ไปที่ MongoDB ซึ่งเป็น Database รูปแบบของ NoSql เพื่อให้สามารถใช้ข้อมูลเหล่านี้ได้ในภายหลัง

## การทำงาน
โดยในการรับส่งข้อมูลนั้นมีจะส่งผ่าน Kafka connect ที่จะถูกเชื่อมต่อกับส่วนต่างๆ ทั้ง MQTT, Prometheus และ MongoDB ซึ่ง Kafka จะเป็นตัวจัดการ ซึ่งจะมี 3 collection สำหรับ MongoDB มีหลักการทำงานดังนี้
* รับข้อมูลจาก iot_frames
* ส่งข้อมูลไปที่ Collection ที่กำหนด โดย ณ ที่นี้ใช้ iot_frames ลงใน Database IOT

File Path: kafka_connect/data/scripts/config/connect-mongodb-iot-frames-sink.json

```json
{
   "name":"iot-frames-mongodb-sink",
   "config":{
      "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
      "tasks.max":1,
      "topics":"iot-frames",
      "connection.uri":"mongodb://devroot:devroot@mongo:27017",
      "database":"iot",
      "collection":"iot_frames",
      "value.converter": "org.apache.kafka.connect.json.JsonConverter",
      "key.converter": "org.apache.kafka.connect.json.JsonConverter",
      "value.converter.schemas.enable": false,
      "key.converter.schemas.enable":false
   }
}
```
ในส่วนเชื่อมต่อที่ 2 เป็นการดึงข้อมูลจาก "iot-aggregate-metric-sensor" ไปที่ Collection "iot_aggregate_metric_sensor"

File Path: kafka_connect/data/scripts/config/connect-mongodb-iot-aggregate-metrics-sensor-sink.json
```json
{
   "name":"iot-aggregate-metrics-sensor-mongodb-sink",
   "config":{
      "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
      "tasks.max":1,
      "topics":"iot-aggregate-metrics-sensor",
      "connection.uri":"mongodb://devroot:devroot@mongo:27017",
      "database":"iot",
      "collection":"iot_aggregate_metrics_sensor",
      "value.converter": "org.apache.kafka.connect.json.JsonConverter",
      "key.converter": "org.apache.kafka.connect.storage.StringConverter",
      "value.converter.schemas.enable": false,
      "key.converter.schemas.enable":false
   }
}
```

ส่วนสุดท้่ายเป็นการดึงข้อมูลจาก Topic "iot-aggregate-metric-place" ไปที่ Collection "iot_aggreagate_metric_place"

File Path: kafka_connect/data/scripts/config/connect-mongodb-iot-aggregate-metrics-place-sink.json

```json
{
   "name":"iot-aggregate-metrics-place-mongodb-sink",
   "config":{
      "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
      "tasks.max":1,
      "topics":"iot-aggregate-metrics-place",
      "connection.uri":"mongodb://devroot:devroot@mongo:27017",
      "database":"iot",
      "collection":"iot_aggregate_metrics_place",
      "value.converter": "org.apache.kafka.connect.json.JsonConverter",
      "key.converter": "org.apache.kafka.connect.storage.StringConverter",
      "value.converter.schemas.enable": false,
      "key.converter.schemas.enable":false
   }
}
```

ในส่วนของข้อมูลที่เป็น Time Serires จะต้องเก็บข้อมูลไปที่ Prometheus โดยจะดึงข้อมูลจาก Topic "iot-metric-time-series" โดย Prometheus ดึงข้อมูลผ่าน HTTP server

File Path: kafka_connect/data/scripts/config/connect-prometheus-sink.json
```json
{
  "name" : "prometheus-connector-sink",
  "config" : {
   "topics":"iot-metrics-time-series",
   "connector.class" : "io.confluent.connect.prometheus.PrometheusMetricsSinkConnector",
   "tasks.max" : "1",
   "confluent.topic.bootstrap.servers":"kafka:9092",
   "prometheus.scrape.url": "http://0.0.0.0:8084/iot-metrics-time-series",
   "prometheus.listener.url": "http://0.0.0.0:8084/iot-metrics-time-series",
   "value.converter": "org.apache.kafka.connect.json.JsonConverter",
   "key.converter": "org.apache.kafka.connect.json.JsonConverter",
   "value.converter.schemas.enable": false,
   "key.converter.schemas.enable":false,
   "reporter.bootstrap.servers": "kafka:9092",
   "reporter.result.topic.replication.factor": "1",
   "reporter.error.topic.replication.factor": "1",
   "behavior.on.error": "log"
  }
}
```


### ข้อจำกัด Prometheus Connector
* ไม่รอบรับ Timestamp: Prometheus ใช้ timestamp จากการดึงข้อมูล (scrape) แต่ละครั้ง ดังนั้น timestamp ใน Kafka records จะถูกละเว้น

* Pull-based Connector: ดึงข้อมูลผ่าน HTTP server บน worker node โดยที่ metric ทั้งหมดจะถูกเผยแพร่ผ่าน HTTP endpoint ที่กำหนดด้วย property prometheus.scrape.url ซึ่งต้องถูกเพิ่มลงในไฟล์การตั้งค่า prometheus.yml

* Metric type: Metrics ส่วนใหญ่จะถูกตีความเป็น gauge type (ตัวเลขที่เพิ่ม / ลด เช่น อุณหภูมิ) ยกเว้น counter type ที่ใช้สำหรับค่าที่เพิ่มขึ้นเพียงอย่างเดียว

* Buffer Limit: Connector จะมี buffer ขนาด 3 ล้าน metric items เพื่อให้ Prometheus มีเวลาเพียงพอในการอ่านและประมวลผลข้อมูลระหว่างการดึงข้อมูลแต่ละครั้ง
