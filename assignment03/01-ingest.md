# MQTT

### Payload

```java
{
  "id": "43245253",
  "name": "iot_sensor_3",
  "place_id": "42343243",
  "date": "2020–09–19T12:19:44.382+0000",
  "timestamp": 1600517984382,
  "payload": {
    "temperature": -1,
    "humidity": 41,
    "pressure": 1023,
    "luminosity": 31322
   }
}
```

## <mark>การส่งข้อมูลจากเซ็นเซอร์ IoT ไปยัง MQTT Broker

> การจำลองเซ็นเซอร์ IoT

> > - เซ็นเซอร์ IoT ถูกจำลองด้วยไมโครเซอร์วิสที่พัฒนาด้วย Spring Boot
> > - ไมโครเซอร์วิสเหล่านี้ใช้ Eclipse Paho MQTT Library เพื่อส่งข้อมูลเทเลเมทรี (เช่น อุณหภูมิ, ความชื้น, ความดัน, และความสว่าง) ไปยัง MQTT broker ชื่อว่า Eclipse Mosquitto
> > - ข้อมูลจะถูกสร้างขึ้นทุกวินาที โดย Callable จะสร้าง payload ของข้อมูลเซ็นเซอร์จำลอง ทำการ serialize เป็น JSON และส่งไปยัง MQTT topic ผ่านไคลเอนต์ MQTT

```java
@Component
public class IoTSensor implements Callable<Void> {

    private static final Logger logger = LoggerFactory.getLogger(IoTSensor.class);

    private final IMqttClient mqttClient;
    private final ObjectMapper objectMapper;

    @Value("${mqtt.server.topic}")
    private String mqttTopic;
    @Value("${sensor.name}")
    private String sensorName;
    @Value("${sensor.id}")
    private String sensorId;
    @Value("${sensor.place.id}")
    private String sensorPlaceId;

    private final Random rnd = new Random();

    /**
     *
     * @param mqttClient
     * @param objectMapper
     */
    @Autowired
    public IoTSensor(final IMqttClient mqttClient, final ObjectMapper objectMapper) {
        this.mqttClient = mqttClient;
        this.objectMapper = objectMapper;
    }

    @Override
    public Void call() throws Exception {
        if (!mqttClient.isConnected()) {
            return null;
        }
        mqttClient.publish(mqttTopic, buildMessage());
        return null;
    }

    /**
     * Build MQTT Message
     *
     * @return
     * @throws JsonProcessingException
     */
    private MqttMessage buildMessage() throws JsonProcessingException {

        //Generate Sensor Values
        int temperature = rnd.ints(-5, 35).findFirst().getAsInt();
        int humidity = rnd.ints(0, 100).findFirst().getAsInt();
        int pressure = rnd.ints(1000, 1030).findFirst().getAsInt();
        int luminosity = rnd.ints(0, 65000).findFirst().getAsInt();

        // Generate Payload
        final SensorDataDTO data = SensorDataDTO.builder()
                .id(sensorId)
                .name(sensorName)
                .placeId(sensorPlaceId)
                .date(new Date())
                .timestamp(new Date().getTime())
                .payload(SensorDataPayloadDTO.builder()
                        .humidity(humidity)
                        .luminosity(luminosity)
                        .pressure(pressure)
                        .temperature(temperature)
                        .build())
                .build();

        final String payload = objectMapper.writeValueAsString(data);

        logger.debug("Payload -> " + payload);

        // Create and configure MQTT Message
        final MqttMessage message = new MqttMessage(payload.getBytes(Charset.forName("UTF-8")));
        message.setQos(1);
        message.setRetained(true);
        return message;
    }

}
```

## <mark>การเชื่อมต่อระหว่าง MQTT Broker กับ Kafka

MQTT Broker (Eclipse Mosquitto)

- Mosquitto ทำหน้าที่เป็น MQTT broker รับข้อมูลจากเซ็นเซอร์ IoT ที่ส่งมาผ่าน MQTT client
- ข้อมูลที่ได้รับจะถูกส่งต่อไปยัง Kafka ผ่าน Kafka Connect โดยใช้ MQTT source connector ที่ถูกตั้งค่าให้รับข้อมูลจาก Mosquitto และส่งต่อไปยัง Kafka topic ที่ชื่อ "iot-frames"

Kafka Connect Configuration

- Kafka Connect ทำหน้าที่เป็นสะพานเชื่อมระหว่าง Mosquitto กับ Kafka
- MQTT source connector ถูกตั้งค่าให้สมัครรับข้อมูลจาก MQTT topics และส่งข้อมูลไปยัง Kafka topic โดยใช้ตัวระบุเซ็นเซอร์ (sensor ID) เป็นคีย์ของข้อมูลใน Kafka
- MQTT source connector ยังใช้ค่าการตั้งค่า เช่น:
- mqtt.topics: ระบุ MQTT topic ที่ connector จะสมัครรับข้อมูล
- kafka.topic: ระบุ Kafka topic ที่จะเก็บข้อมูล
- mqtt.qos: ระดับ QoS ของ MQTT ที่จะใช้ในการสมัครรับข้อมูล

## <mark>การเพิ่ม security ในการใช้งาน MQTT

โดยปกติแล้วการส่งข้อมูลของ client บน MQTT สามารถส่งได้เลยผ่านการ subscribe topic ที่ต้องการ แต่หากต้องการเพิ่ม security สามารถทำได้โดยการเพิ่ม authentication ให้ client เพื่อให้ client ต้องกรอก username และ password ให้ถูกต้องก่อน ถึงจะสามารถส่งข้อมูลได้ ซึ่งการสร้าง username และ password นั้น สามารถสร้างได้ที่ broker ที่จะกำหนดได้เลยว่า username และ password ที่สามารถเข้ามาเชื่อมต่อส่งข้อมูล มีอะไรบ้าง ทำให้การส่งข้อมูลบน MQTT มีความ security มากขึ้น

## <mark>สรุป

MQTT เป็นโปรโตคอลที่ใช้สำหรับการสื่อสารข้อมูลแบบเบาและรวดเร็วในระบบ IoT โดยในโปรเจกต์นี้ เซ็นเซอร์ IoT จะส่งข้อมูลเทเลเมทรีไปยัง Eclipse Mosquitto ซึ่งทำหน้าที่เป็น MQTT broker จากนั้น Kafka Connect ที่ใช้ MQTT source connector จะรับข้อมูลจาก Mosquitto และส่งต่อไปยัง Kafka เพื่อจัดเก็บและประมวลผลต่อไป.
