# Analyze and make aggregations.

## <mark>อะไรคือการทำ Analyze และ Aggregations ?

## <mark>อะไรคือการทำ Analyze และ Aggregations ?

การวิเคราะห์และการทำการรวมข้อมูล (Analyze and Make Aggregations): ขั้นตอนที่สำคัญในการทำให้ข้อมูลที่ได้จากเซ็นเซอร์เป็นประโยชน์และใช้งานได้ง่ายขึ้น โดยการสรุปข้อมูลที่ได้ในช่วงเวลาต่างๆ เพื่อนำเสนอข้อมูลสถิติหรือข้อมูลเชิงลึก เช่น ค่าเฉลี่ย ค่าสูงสุด หรือต่ำสุด ของอุณหภูมิ ความชื้น แสงสว่าง ฯลฯ ในช่วงเวลาที่กำหนด

### ตัวอย่างง่าย ๆ 🦺

ลองนึกภาพว่ามีเซ็นเซอร์ที่เก็บข้อมูลอุณหภูมิทุกๆ วินาที ระบบจะจับข้อมูลที่เข้ามาทุกๆ 5 นาทีแล้วคำนวณค่าเฉลี่ยออกมา หากในช่วง 5 นาทีแรก ค่าเฉลี่ยอุณหภูมิเป็น 25°C และในช่วง 5 นาทีถัดไปค่าเฉลี่ยเป็น 27°C คุณจะได้ข้อมูลว่าอุณหภูมิกำลังเพิ่มขึ้น ซึ่งจะเป็นประโยชน์ในการตัดสินใจ เช่น เปิดเครื่องปรับอากาศ เป็นต้น

## <mark>IoT Processor Kafka Streams Config

การตั้งค่าการใช้งาน Kafka Streams เพื่อรองรับการประมวลผลข้อมูลจากเซ็นเซอร์โดยใช้บริการที่ชื่อว่า KafkaStreamsConfig ซึ่งมีหน้าที่หลักในการจัดการการไหลของข้อมูล (data stream) ที่มาจาก Kafka topic และใช้โปรเซสเซอร์ต่าง ๆ ในการประมวลผลข้อมูลเหล่านั้น

บริการ iot-processor ซึ่งมีตัวประมวลผลสามตัวได้แก่

1. Aggregate Metrics By Sensor Processor
2. Aggregate Metrics By Place Processor
3. MetricsTimeSeriesProcessor

```java
@Configuration
@EnableKafka
@EnableKafkaStreams
public class KafkaStreamsConfig {

    private static final Logger logger = LoggerFactory.getLogger(KafkaStreamsConfig.class);

    @Value("${kafka.topic.input}")
    private String inputTopic;

    /**
     * Aggregate Metrics By Sensor Processor
     */
    @Autowired
    private AggregateMetricsBySensorProcessor aggregateMetricsBySensorProcessor;

    /**
     * Aggregate Metrics By Place Processor
     */
    @Autowired
    private AggregateMetricsByPlaceProcessor aggregateMetricsByPlaceProcessor;

    /**
     * Metrics Time Series Processor
     */
    @Autowired
    private MetricsTimeSeriesProcessor metricsTimeSeriesProcessor;

    /**
     * Provide KStreams Configs
     *
     * @param kafkaProperties
     * @return
     */
    @Bean(name = KafkaStreamsDefaultConfiguration.DEFAULT_STREAMS_CONFIG_BEAN_NAME)
    public KafkaStreamsConfiguration provideKStreamsConfigs(final KafkaProperties kafkaProperties) {
        Map<String, Object> config = new HashMap<>();
        config.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, kafkaProperties.getBootstrapServers());
        config.put(StreamsConfig.APPLICATION_ID_CONFIG, kafkaProperties.getClientId());
        config.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, SensorKeySerde.class.getName());
        config.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, SensorDataSerde.class.getName());
        config.put(StreamsConfig.DEFAULT_DESERIALIZATION_EXCEPTION_HANDLER_CLASS_CONFIG, LogAndContinueExceptionHandler.class);
        return new KafkaStreamsConfiguration(config);
    }

    /**
     * Provide KStream
     *
     * @param kStreamBuilder
     * @return
     */
    @Bean
    public KStream<SensorKeyDTO, SensorDataDTO> provideKStream(StreamsBuilder kStreamBuilder) {
        final KStream<SensorKeyDTO, SensorDataDTO> stream = kStreamBuilder.stream(inputTopic);
        logger.debug("Start Aggregate Metrics By Sensor Processor Stream");
        aggregateMetricsBySensorProcessor.process(stream);
        logger.debug("Start Aggregate Metrics By Place Processor Stream");
        aggregateMetricsByPlaceProcessor.process(stream);
        logger.debug("Metrics Time Series Processor Stream");
        metricsTimeSeriesProcessor.process(stream);
        return stream;
    }

}
```

<<<<<<< HEAD
<<<<<<< HEAD

## <mark>First processor "Aggregate Metrics By Sensor Processor"

=======

## First processor "Aggregate Metrics By Sensor Processor"

=======

## <mark>First processor "Aggregate Metrics By Sensor Processor"

> > > > > > > 8115ca5 (:memo: : adjust as5 format.)

> > > > > > > 4c6e620 (update readme assignment 5)

ตัวประมวลผลตัวแรกจะทำการรวมข้อมูลตาม sensor id โดยใช้ rotating time window ที่มีช่วงเวลา 5 นาที

โดยมีวัตถุประสงค์: เพื่อคำนวณค่าเฉลี่ยสำหรับตัวแปรต่างๆ เช่น อุณหภูมิ, ความชื้น, ความดัน และความสว่าง โดยใช้ข้อมูลจากเซ็นเซอร์แต่ละตัว

ตัวประมวลผลจะสร้างโมเดลข้อมูลใหม่ที่มีโครงสร้าง (schema) ต่างจากเดิม ซึ่งจำเป็นต้องตั้งค่า SerDe (Serializer/Deserializer) ให้เหมาะสมเพื่อใช้ในการ serialize และ deserialize ข้อมูลใน Kafka

<<<<<<< HEAD

# <<<<<<< HEAD

> > > > > > > 4c6e620 (update readme assignment 5)
> > > > > > > ผลลัพธ์: เมื่อหน้าต่างเวลาถูกปิด ตัวประมวลผลจะสร้างเรคคอร์ดที่ประกอบด้วยข้อมูลการรวมประมาณ 300 รายการต่อเซ็นเซอร์ และบันทึกค่าเฉลี่ยที่คำนวณได้
> > > > > > > ผลลัพธ์จะถูกบันทึกลงใน Kafka topic ที่ชื่อ iot-aggregate-metrics-by-sensor
> > > > > > > =======
> > > > > > > ผลลัพธ์: เมื่อหน้าต่างเวลาถูกปิด ตัวประมวลผลจะสร้างเรคคอร์ดที่ประกอบด้วยข้อมูลการรวมประมาณ 300 รายการต่อเซ็นเซอร์ และบันทึกค่าเฉลี่ยที่คำนวณได้
> > > > > > > ผลลัพธ์จะถูกบันทึกลงใน Kafka topic ที่ชื่อ iot-aggregate-metrics-by-sensor
> > > > > > > 8115ca5 (:memo: : adjust as5 format.)

```java
@Component
public class AggregateMetricsBySensorProcessor {

    private static final Logger logger = LoggerFactory.getLogger(AggregateMetricsBySensorProcessor.class);

    private final static int WINDOW_SIZE_IN_MINUTES = 5;
    private final static String WINDOW_STORE_NAME = "aggregate-metrics-by-sensor-tmp";

    /**
     * Agg Metrics Sensor Topic Output
     */
    @Value("${kafka.topic.aggregate-metrics-sensor}")
    private String aggMetricsSensorOutput;

    /**
     *
     * @param stream
     */
    public void process(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        buildAggregateMetricsBySensor(stream)
                .to(aggMetricsSensorOutput, Produced.with(String(), new SensorAggregateMetricsSensorSerde()));
    }

    /**
     * Build Aggregate Metrics By Sensor Stream
     *
     * @param stream
     * @return
     */
    private KStream<String, SensorAggregateSensorMetricsDTO> buildAggregateMetricsBySensor(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        return stream
                .map((key, val) -> new KeyValue<>(val.getId(), val))
                .groupByKey(Grouped.with(String(), new SensorDataSerde()))
                .windowedBy(TimeWindows.of(Duration.ofMinutes(WINDOW_SIZE_IN_MINUTES)).grace(Duration.ofMillis(0)))
                .aggregate(SensorAggregateSensorMetricsDTO::new,
                        (String k, SensorDataDTO v, SensorAggregateSensorMetricsDTO va) -> aggregateData(v, va),
                         buildWindowPersistentStore()
                )
                .suppress(Suppressed.untilWindowCloses(unbounded()))
                .toStream()
                .map((key, value) -> KeyValue.pair(key.key(), value));
    }

    /**
     * Build Window Persistent Store
     *
     * @return
     */
    private Materialized<String, SensorAggregateSensorMetricsDTO, WindowStore<Bytes, byte[]>> buildWindowPersistentStore() {
        return Materialized
                .<String, SensorAggregateSensorMetricsDTO, WindowStore<Bytes, byte[]>>as(WINDOW_STORE_NAME)
                .withKeySerde(String())
                .withValueSerde(new SensorAggregateMetricsSensorSerde());
    }

    /**
     * Aggregate Data
     *
     * @param v
     * @param va
     * @return
     */
    private SensorAggregateSensorMetricsDTO aggregateData(final SensorDataDTO v, final SensorAggregateSensorMetricsDTO va) {
        // Sensor Data
        va.setId(v.getId());
        // Sensor Data
        va.setId(v.getId());
        va.setName(v.getName());
        // Start Agg
        if (va.getStartAgg() == null) {
            final Date startAggAt = new Date();
            va.setStartAgg(startAggAt);
            va.setStartAggTm(startAggAt.getTime());
        }
        va.setCountMeasures(va.getCountMeasures() + 1);
        // Temperature
        va.setSumTemperature(va.getSumTemperature() + v.getPayload().getTemperature());
        va.setAvgTemperature(va.getSumTemperature() / va.getCountMeasures()); // Humidity
        // Humidity
        va.setSumHumidity(va.getSumHumidity() + v.getPayload().getHumidity());
        va.setAvgHumidity(va.getSumHumidity() / va.getCountMeasures()); // Luminosity
        // Luminosity
        va.setSumLuminosity(va.getSumLuminosity() + v.getPayload().getLuminosity());
        va.setAvgLuminosity(va.getSumLuminosity() / va.getCountMeasures()); // Pressure
        // Pressure
        va.setSumPressure(va.getSumPressure() + v.getPayload().getPressure());
        va.setAvgPressure(va.getSumPressure() / va.getCountMeasures());

        // End Agg
        final Date endAggAt = new Date();
        va.setEndAgg(endAggAt);
        va.setEndAggTm(endAggAt.getTime());
        return va;
    }

}
```

<<<<<<< HEAD
<<<<<<< HEAD

## <mark>Second processor "Aggregate Metrics By Place Processor"

ตัวประมวลผลตัวที่สองจะทำงานคล้ายกับตัวแรก แต่จะทำการรวมข้อมูลตาม place identifier แทนที่จะเป็น sensor id

# โดยมีวัตถุประสงค์:

## Second processor "Aggregate Metrics By Place Processor"

=======

## <mark>Second processor "Aggregate Metrics By Place Processor"

> > > > > > > 8115ca5 (:memo: : adjust as5 format.)

ตัวประมวลผลตัวที่สองจะทำงานคล้ายกับตัวแรก แต่จะทำการรวมข้อมูลตาม place identifier แทนที่จะเป็น sensor id

โดยมีวัตถุประสงค์:
<<<<<<< HEAD

> > > > > > > 4c6e620 (update readme assignment 5)
> > > > > > > เพื่อคำนวณค่าเฉลี่ยตามสถานที่ (place) โดยการรวมข้อมูลจากเซ็นเซอร์หลายตัวที่อยู่ในสถานที่เดียวกัน
> > > > > > > =======
> > > > > > > เพื่อคำนวณค่าเฉลี่ยตามสถานที่ (place) โดยการรวมข้อมูลจากเซ็นเซอร์หลายตัวที่อยู่ในสถานที่เดียวกัน
> > > > > > > 8115ca5 (:memo: : adjust as5 format.)

เช่นเดียวกับตัวประมวลผลแรก จะต้องตั้งค่า SerDe ให้เหมาะสมเพื่อรองรับ schema ที่แตกต่างกัน

ผลลัพธ์จะถูกบันทึกลงใน Kafka topic ที่ชื่อ iot-aggregate-metrics-place

```java
@Component
public class AggregateMetricsByPlaceProcessor {

    private static final Logger logger = LoggerFactory.getLogger(AggregateMetricsByPlaceProcessor.class);

    private final static int WINDOW_SIZE_IN_MINUTES = 5;
    private final static String WINDOW_STORE_NAME = "aggregate-metrics-by-place-tmp";

    /**
     * Agg Metrics Place Topic Output
     */
    @Value("${kafka.topic.aggregate-metrics-place}")
    private String aggMetricsPlaceOutput;

    /**
     *
     * @param stream
     */
    public void process(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        buildAggregateMetrics(stream)
                .to(aggMetricsPlaceOutput, Produced.with(String(), new SensorAggregateMetricsPlaceSerde()));
    }

    /**
     * Build Aggregate Metrics Stream
     *
     * @param stream
     * @return
     */
    private KStream<String, SensorAggregatePlaceMetricsDTO> buildAggregateMetrics(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        return stream
                .map((key, val) -> new KeyValue<>(val.getPlaceId(), val))
                .groupByKey(Grouped.with(String(), new SensorDataSerde()))
                .windowedBy(TimeWindows.of(Duration.ofMinutes(WINDOW_SIZE_IN_MINUTES)).grace(Duration.ofMillis(0)))
                .aggregate(SensorAggregatePlaceMetricsDTO::new,
                        (String k, SensorDataDTO v, SensorAggregatePlaceMetricsDTO va) -> aggregateData(v, va),
                        buildWindowPersistentStore()
                )
                .suppress(Suppressed.untilWindowCloses(unbounded()))
                .toStream()
                .map((key, value) -> KeyValue.pair(key.key(), value));
    }

    /**
     * Build Window Persistent Store
     *
     * @return
     */
    private Materialized<String, SensorAggregatePlaceMetricsDTO, WindowStore<Bytes, byte[]>> buildWindowPersistentStore() {
        return Materialized
                .<String, SensorAggregatePlaceMetricsDTO, WindowStore<Bytes, byte[]>>as(WINDOW_STORE_NAME)
                .withKeySerde(String())
                .withValueSerde(new SensorAggregateMetricsPlaceSerde());
    }

    /**
     * Aggregate Data
     *
     * @param v
     * @param va
     * @return
     */
    private SensorAggregatePlaceMetricsDTO aggregateData(final SensorDataDTO v, final SensorAggregatePlaceMetricsDTO va) {
        va.setPlaceId(v.getId());
        // Start Agg
        if (va.getStartAgg() == null) {
            final Date startAggAt = new Date();
            va.setStartAgg(startAggAt);
            va.setStartAggTm(startAggAt.getTime());
        }
        va.setCountMeasures(va.getCountMeasures() + 1);
        // Temperature
        va.setSumTemperature(va.getSumTemperature() + v.getPayload().getTemperature());
        va.setAvgTemperature(va.getSumTemperature() / va.getCountMeasures()); // Humidity
        // Humidity
        va.setSumHumidity(va.getSumHumidity() + v.getPayload().getHumidity());
        va.setAvgHumidity(va.getSumHumidity() / va.getCountMeasures()); // Luminosity
        // Luminosity
        va.setSumLuminosity(va.getSumLuminosity() + v.getPayload().getLuminosity());
        va.setAvgLuminosity(va.getSumLuminosity() / va.getCountMeasures()); // Pressure
        // Pressure
        va.setSumPressure(va.getSumPressure() + v.getPayload().getPressure());
        va.setAvgPressure(va.getSumPressure() / va.getCountMeasures());

        // End Agg
        final Date endAggAt = new Date();
        va.setEndAgg(endAggAt);
        va.setEndAggTm(endAggAt.getTime());
        return va;
    }

}
```

<<<<<<< HEAD
<<<<<<< HEAD

## <mark>Third processor "Metrics Time Series Processor"

ตัวประมวลผลตัวที่สามมีหน้าที่ในการแปลงข้อมูลให้อยู่ในรูปแบบที่เข้ากันได้กับ Prometheus ซึ่งเป็นระบบเก็บข้อมูลเมตริกที่นิยมใช้ในงานตรวจสอบระบบ

# โดยมีวัตถุประสงค์:

## Third processor "Metrics Time Series Processor"

=======

## <mark>Third processor "Metrics Time Series Processor"

> > > > > > > 8115ca5 (:memo: : adjust as5 format.)

ตัวประมวลผลตัวที่สามมีหน้าที่ในการแปลงข้อมูลให้อยู่ในรูปแบบที่เข้ากันได้กับ Prometheus ซึ่งเป็นระบบเก็บข้อมูลเมตริกที่นิยมใช้ในงานตรวจสอบระบบ

โดยมีวัตถุประสงค์:
<<<<<<< HEAD

> > > > > > > 4c6e620 (update readme assignment 5)
> > > > > > > เปลี่ยนข้อมูลเป็น schema ที่เข้ากับ Prometheus โดยใช้ชื่อ, sensor id, และ place identifier เป็นมิติ (dimension) ของข้อมูล และใช้ payload เป็นค่าที่สามารถแสดงผลในรูปแบบ time series บน Grafana ได้
> > > > > > > =======
> > > > > > > เปลี่ยนข้อมูลเป็น schema ที่เข้ากับ Prometheus โดยใช้ชื่อ, sensor id, และ place identifier เป็นมิติ (dimension) ของข้อมูล และใช้ payload เป็นค่าที่สามารถแสดงผลในรูปแบบ time series บน Grafana ได้
> > > > > > > 8115ca5 (:memo: : adjust as5 format.)

ข้อมูลนี้จะถูกบันทึกลงใน Kafka topic ที่ชื่อ iot-metrics-time-series ซึ่ง Prometheus จะเข้ามาดึงข้อมูลจาก topic นี้เพื่อนำไปแสดงผลเป็นเมตริก

```java
@Component
public class MetricsTimeSeriesProcessor {

    private static final Logger logger = LoggerFactory.getLogger(MetricsTimeSeriesProcessor.class);

    private final static String SENSOR_TIME_SERIE_NAME = "sample_sensor_metric";
    private final static String SENSOR_TIME_SERIE_TYPE = "sensor";

    /**
     * Metrics Time Series
     */
    @Value("${kafka.topic.metrics-time-series}")
    private String metricTimeSeriesOutput;

    /**
     *
     * @param stream
     */
    public void process(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        stream
                .map((key, val) -> new KeyValue<>(val.getId(), buildSensorTimeSerieMetric(val)))
                .to(metricTimeSeriesOutput, Produced.with(String(), new SensorTimeSerieMetricSerde()));
    }

    /**
     * Build Sensor Time Serie Meter
     *
     * @param sensorData
     * @return
     */
    private SensorTimeSerieMetricDTO buildSensorTimeSerieMetric(final SensorDataDTO sensorData) {
        return SensorTimeSerieMetricDTO.builder()
                .name(SENSOR_TIME_SERIE_NAME)
                .timestamp(new Date().getTime())
                .type(SENSOR_TIME_SERIE_TYPE)
                .dimensions(SensorTimeSerieMetricDimensionsDTO.builder()
                        .placeId(sensorData.getPlaceId())
                        .sensorId(sensorData.getId())
                        .sensorName(sensorData.getName())
                        .build())
                .values(SensorTimeSerieMetricValuesDTO.builder()
                        .humidity((double) sensorData.getPayload().getHumidity())
                        .luminosity((double) sensorData.getPayload().getLuminosity())
                        .pressure((double) sensorData.getPayload().getPressure())
                        .temperature((double) sensorData.getPayload().getTemperature())
                        .build())
                .build();
    }

}
```

<<<<<<< HEAD
<<<<<<< HEAD

## <mark>ความสำคัญของ SerDe

# การใช้ SerDe (Serializer/Deserializer) เป็นสิ่งสำคัญ เนื่องจากข้อมูลที่สร้างขึ้นและประมวลผลต้องถูกแปลงให้อยู่ในรูปแบบที่ Kafka สามารถจัดเก็บและส่งต่อได้ และยังต้องสามารถนำข้อมูลนี้ไปใช้ในส่วนที่ต้องการได้อย่างถูกต้องอีกด้วย

## ความสำคัญของ SerDe

การใช้ SerDe (Serializer/Deserializer) เป็นสิ่งสำคัญ เนื่องจากข้อมูลที่สร้างขึ้นและประมวลผลต้องถูกแปลงให้อยู่ในรูปแบบที่ Kafka สามารถจัดเก็บและส่งต่อได้ และยังต้องสามารถนำข้อมูลนี้ไปใช้ในส่วนที่ต้องการได้อย่างถูกต้องอีกด้วย

> > > > > > > # 4c6e620 (update readme assignment 5)

## <mark>ความสำคัญของ SerDe

การใช้ SerDe (Serializer/Deserializer) เป็นสิ่งสำคัญ เนื่องจากข้อมูลที่สร้างขึ้นและประมวลผลต้องถูกแปลงให้อยู่ในรูปแบบที่ Kafka สามารถจัดเก็บและส่งต่อได้ และยังต้องสามารถนำข้อมูลนี้ไปใช้ในส่วนที่ต้องการได้อย่างถูกต้องอีกด้วย

> > > > > > > 8115ca5 (:memo: : adjust as5 format.)
