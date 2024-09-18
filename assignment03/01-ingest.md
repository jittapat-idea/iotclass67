# MQTT

## iot-sensor-1

IoT Sensor 1 เป็น IoT ที่ได้มาจากการ compose up docker ใน Server ตัวมันเอง ซึ่งมาจาก image IOT senser ซึ่งจะถูกสุ่มค่าด้วย Spring Boot

## iot-sensor-2

IoT Sensor 2 เป็น IoT ที่ได้มาจากการ compose up docker ใน PC ตัวนอกที่ได้ทำการส่งข้อมูลไปยัง Server ใช้ image IOT senser และสุ่มค่าด้วย Spring Boot เช่นเดียวกับ iot-sensor-1

เมื่อ Run ใน Local จะต้องแก้ไขไฟล์ pom.xml เพื่อจัดการ maven ให้ตรงกับ Architecture ของเครื่องของเรา

### pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.dreamsoftware</groupId>
    <artifactId>iotframesingest</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>iot_sensor</name>
    <description>Simulate IoT Sensor</description>

    <properties>
        <java.version>1.8</java.version>
        <eclipse-paho-mqtt.version>1.2.0</eclipse-paho-mqtt.version>
        <jackson-bind.version>2.9.8</jackson-bind.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.eclipse.paho</groupId>
            <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
            <version>${eclipse-paho-mqtt.version}</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${jackson-bind.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <finalName>iot_sensor</finalName>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>1.18.12</version>
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>

            <plugin>
                    <groupId>com.spotify</groupId>
                    <artifactId>dockerfile-maven-plugin</artifactId>
                    <version>1.4.6</version>
                    <dependencies>
                        <dependency>
                            <groupId>com.github.jnr</groupId>
                            <artifactId>jnr-unixsocket</artifactId>
                            <version>0.38.14</version>
                        </dependency>
                    </dependencies>
            </plugin>
        </plugins>
    </build>

</project>
```

จากนั้นต้องแก้ไขการเชื่อมต่อกับ MQTT ใหม่ โดยจะต้องแก้เป็นการส่งไปให้กับ Server ของเราผ่าน IP ของเครื่อง Server ของเราโดยจะส่งไปให้กับ
Kafka-connect

### connect-mqtt-source.json
```json
{
    "name": "mqtt-source",
    "config": {
        "connector.class": "io.confluent.connect.mqtt.MqttSourceConnector",
        "tasks.max": 1,
        "mqtt.server.uri": "tcp://172.16.46.11:1883",
        "mqtt.topics": "iot-frames",
        "mqtt.username": "kafka-connect",
        "mqtt.password": "1q2w3e4r",
        "mqtt.qos": 1,
        "kafka.topic": "iot-frames",
        "value.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "key.converter": "org.apache.kafka.connect.json.JsonConverter",
        "key.converter.schemas.enable":false,
        "converter.encoding": "UTF-8",
        "value.converter.schemas.enable": false,
        "confluent.topic.bootstrap.servers": "kafka:9092",
        "confluent.topic.replication.factor": 1,
        "transforms":"createMap,createKey,extractString",
        "transforms.createMap.type":"org.apache.kafka.connect.transforms.HoistField$Value",
        "transforms.createMap.field":"id",
        "transforms.createKey.type":"org.apache.kafka.connect.transforms.ValueToKey",
        "transforms.createKey.fields":"id",
        "transforms.extractString.type":"org.apache.kafka.connect.transforms.ExtractField$Value",
        "transforms.extractString.field":"id"
    }
}

{
    "name": "mqtt-source",
    "config": {
        "connector.class": "io.confluent.connect.mqtt.MqttSourceConnector",
        "tasks.max": 1,
        "mqtt.server.uri": "tcp://ipของเครื่องเรา:1883",
        "mqtt.topics": "iot-frames",
        "mqtt.username": "kafka-connect",
        "mqtt.password": "XXXXX",
        "mqtt.qos": 1,
        "kafka.topic": "iot-frames",
        "value.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "key.converter": "org.apache.kafka.connect.json.JsonConverter",
        "key.converter.schemas.enable":false,
        "converter.encoding": "UTF-8",
        "value.converter.schemas.enable": false,
        "confluent.topic.bootstrap.servers": "kafka:9092",
        "confluent.topic.replication.factor": 1,
        "transforms":"createMap,createKey,extractString",
        "transforms.createMap.type":"org.apache.kafka.connect.transforms.HoistField$Value",
        "transforms.createMap.field":"id",
        "transforms.createKey.type":"org.apache.kafka.connect.transforms.ValueToKey",
        "transforms.createKey.fields":"id",
        "transforms.extractString.type":"org.apache.kafka.connect.transforms.ExtractField$Value",
        "transforms.extractString.field":"id"
    }
}
```

### applicaiton.properties
```config
# MQTT Config
mqtt.server.url=tcp://172.16.46.11:1883
mqtt.server.topic=iot-frames

# Log
logging.level.com.dreamsoftware.iotframesingest=DEBUG
```

จากนั้นจะทำการ Compile Maven เพื่อให้ IOT-sensor ส่งไปให้กับ Kafka-connect ให้ตรงกับ IP address server ของเรา
```bash
# Use maven complie iot sensor for arm
docker run -it --rm --name my-maven-project -v "$(pwd)":/usr/src/mymaven -w /usr/src/mymaven arm64v8/maven:3.8-jdk-8 mvn clean install
```
## iot-sensor-3-10

IoT Sensor 3 ถึง 10 เป็น IoT ที่ได้ข้อมูลจาก sensor จริงๆ ผ่าน board cumcuber โดยตัวที่ 3 จะเป็นค่าที่ได้จาก board ของกลุ่มเราเอง ตัวที่ 4 - 10 จะเป็นค่าที่ได้จาก board ของกลุ่มอื่นๆ ที่ส่งมาให้เราอีก
