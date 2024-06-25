# IoT Docker compose

## How to start docker compose

> "Note" 🚨 สิ่งแรกที่ต้องทำ คือ การเข้าไปในตำแหน่งของ docker-compose.yml ภายใน Project Directory ก่อน จึงจะสามารถ start docker compose ได้

### การ Start Docker Compose สามารถทำได้ วิธี

1.  Basic Start

```bash
docker compose up
```

> การรันแบบนี้จะขึ้น log ของ Container ทุกตัวพร้อมกัน หากอยากดูทีละตัวให้ใช้ command ที่หัวข้อ Command เพิ่มเติม

2.  Start with "Deamon" (รันแบคกราวน์)

```bash
docker compose up -d
```

### Command เพิ่มเติม

บางครั้งเราก็ไม่อยากดู log ของทุก Container พร้อม ๆ กัน
ซึ่งเราสามารถดู log การทำงานของแต่ละ Container ภายใต้ Compose ได้โดยใช้

```bash
docker compose logs -f "container_name" # formation
docker compose logs -f mongo # example
```

> ➕ ดู log ของ Container ทุกตัว

```bash
docker compose logs -f
```

## Error we found

## How to solve the problems?

## Output

- [ ] IoT Sensor - Dashboards - Grafana
- [ ] UI for Apache Ka
- [ ] Mongo Expr
- [ ] Node Expor
- [ ] Prometheus Time Series Collection and Processing Ser
- [ ] Prometheus Pushgateway
- [ ] ZooNavigator

### IoT Sensor - Dashboards - Grafana URL

### UI for Apache Kafka

### Mongo Express

### Node Exporter

### Prometheus Time Series Collection and Processing Server

### Prometheus Pushgateway

### ZooNavigator
