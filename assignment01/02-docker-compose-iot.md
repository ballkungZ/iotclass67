# IoT Docker compose
<!-- > ให้นำไฟล์ docker-compose.yaml มาอธิบายว่า แต่ละส่วนคืออะไร โดยใช้การ comment ในไฟล์ docker-compose.yaml -->
### docker-compose.yaml
```yaml
services:
    zookeeper:
        image: confluentinc/cp-zookeeper
        container_name: zookeeper
        restart: unless-stopped
        volumes:
        - zookeeper-data:/var/lib/zookeeper/data
        - zookeeper-log:/var/lib/zookeeper/log
        environment:
        ZOOKEEPER_CLIENT_PORT: 2181
        ZOOKEEPER_LOG4J_ROOT_LOGLEVEL: INFO
        ZOOKEEPER_LOG4J_PROP: INFO,ROLLINGFILE
        ZOOKEEPER_LOG_MAXFILESIZE: 10MB
        ZOOKEEPER_LOG_MAXBACKUPINDEX: 10
        ZOOKEEPER_SNAP_COUNT: 10
        ZOOKEEPER_AUTOPURGE_SNAP_RETAIN_COUNT: 10
        ZOOKEEPER_AUTOPURGE_PURGE_INTERVAL: 3
    
    kafka:
        image: confluentinc/cp-kafka
        container_name: kafka
        volumes:
        - kafka-data:/var/lib/kafka
        restart: unless-stopped
        environment:
        KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
        KAFKA_NUM_PARTITIONS: 1
        KAFKA_COMPRESSION_TYPE: gzip
        KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
        KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
        KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
        KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
        KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'
        links:
        - zookeeper

    kafka-rest-proxy:
        image: confluentinc/cp-kafka-rest:latest
        container_name: kafka-rest-proxy
        environment:
        KAFKA_REST_ZOOKEEPER_CONNECT: zookeeper:2181
        KAFKA_REST_LISTENERS: http://0.0.0.0:8082/
        KAFKA_REST_HOST_NAME: kafka-rest-proxy
        KAFKA_REST_BOOTSTRAP_SERVERS: kafka:9092
        restart: unless-stopped
        ports:
        - "9999:8082"
        depends_on:
        - zookeeper
        - kafka

    kafka-connect:
        image: confluentinc/cp-kafka-connect:latest
        hostname: kafka-connect
        container_name: kafka-connect
        environment:
        CONNECT_BOOTSTRAP_SERVERS: "kafka:9092"
        CONNECT_GROUP_ID: kafka-connect-group
        CONNECT_CONFIG_STORAGE_TOPIC: kafka-connect-meta-configs
        CONNECT_OFFSET_STORAGE_TOPIC: kafka-connect-meta-offsets
        CONNECT_STATUS_STORAGE_TOPIC: kafka-connect-meta-status
        CONNECT_KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter
        CONNECT_VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter
        CONNECT_REST_ADVERTISED_HOST_NAME: "kafka-connect"
        CONNECT_REST_PORT: 8083
        CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: "1"
        CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: "1"
        CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: "1"
        CONNECT_PLUGIN_PATH: "/usr/share/java,/data/connectors/"
        CONNECT_LOG4J_ROOT_LOGLEVEL: "INFO"
        restart: unless-stopped
        volumes:
        - ./kafka_connect/data:/data
        command: 
        - bash 
        - -c 
        - |
            echo "Launching Kafka Connect worker"
            /etc/confluent/docker/run & 
            #
            echo "Waiting for Kafka Connect to start listening on http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors ⏳"
            while [ $$(curl -s -o /dev/null -w %{http_code} http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors) -ne 200 ] ; do 
            echo -e $$(date) " Kafka Connect listener HTTP state: " $$(curl -s -o /dev/null -w %{http_code} http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors) " (waiting for 200)"
            sleep 5 
            done
            nc -vz $$CONNECT_REST_ADVERTISED_HOST_NAME $$CONNECT_REST_PORT
            echo -e "\n--\n+> Creating Kafka Connect MongoDB sink Current PATH ($$PWD)"
            /data/scripts/create_mongo_sink.sh 
            echo -e "\n--\n+> Creating MQTT Source Connect Current PATH ($$PWD)"
            /data/scripts/create_mqtt_source.sh
            echo -e "\n--\n+> Creating Kafka Connect Prometheus sink Current PATH ($$PWD)"
            /data/scripts/create_prometheus_sink.sh
            sleep infinity
        depends_on:
        - zookeeper
        - kafka
    
    mosquitto:
        image: eclipse-mosquitto:latest
        hostname: mosquitto
        container_name: mosquitto
        restart: unless-stopped
        ports:
        - "1883:1883"
        - "9001:9001"
        volumes:
        - ./mosquitto/config:/mosquitto/config
        - ./mosquitto/data:/mosquitto/data
        - ./mosquitto/log:/mosquitto/log
    
    mongo:
        image: mongo:4.4.20
        container_name: mongo
        env_file:
        - .env
        restart: unless-stopped
        environment:
        - MONGO_INITDB_ROOT_USERNAME=${MONGO_ROOT_USER}
        - MONGO_INITDB_ROOT_PASSWORD=${MONGO_ROOT_PASSWORD}
        - MONGO_INITDB_DATABASE=${MONGO_DB}

    grafana:
        image: grafana/grafana:latest-ubuntu
        container_name: grafana
        user: '0'
        volumes:
        - ./grafana/data:/var/lib/grafana
        - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
        - ./grafana/datasources:/etc/grafana/provisioning/datasources
        - ./grafana/data/plugins:/var/lib/grafana/plugins

        environment:
        - GF_SECURITY_ADMIN_USER=${ADMIN_USER:-admin}
        - GF_SECURITY_ADMIN_PASSWORD=${ADMIN_PASSWORD:-admin}
        - GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-worldmap-panel,grafana-piechart-panel
        - GF_USERS_ALLOW_SIGN_UP=false
        - GF_SECURITY_ANGULAR_SUPPORT_ENABLED=True
        - GF_FEATURE_TOGGLES_ANGULARDEPRECATIONUI=False
        restart: unless-stopped
        links:
        - prometheus
        ports:
        - '8085:3000'

    prometheus:
        image: prom/prometheus:latest
        container_name: prometheus
        volumes:
        - ./prometheus/:/etc/prometheus/
        - prometheus_data:/prometheus
        command:
        - '--config.file=/etc/prometheus/prometheus.yml'
        - '--storage.tsdb.path=/prometheus'
        - '--web.console.libraries=/etc/prometheus/console_libraries'
        - '--web.console.templates=/etc/prometheus/consoles'
        - '--storage.tsdb.retention.time=200h'
        - '--web.enable-lifecycle'
        restart: unless-stopped
        ports:
        - '8086:9090'

    iot-processor:
        image: ssanchez11/iot_processor:0.0.1-SNAPSHOT
        container_name: iot-processor
        restart: unless-stopped
        ports:
        - '8080:8080'
        depends_on:
        kafka-connect:
            condition: service_started
            restart: true

    iot_sensor_1:
        image: ssanchez11/iot_sensor:0.0.1-SNAPSHOT
        build:
        context: ./microservices/iot_sensor
        args:
            - MQTT_SERVER=${IOT_SENSOR_1_ID}
        container_name: iot_sensor_1
        restart: unless-stopped
        environment:
        - sensor.id=${IOT_SENSOR_1_ID}
        - sensor.name=${IOT_SENSOR_1_NAME}
        - sensor.place.id=${IOT_SENSOR_1_PLACE_ID}
        - sensor.mqtt.username=${IOT_SENSOR_1_USERNAME}
        - sensor.mqtt.password=${IOT_SENSOR_1_PASSWORD}
        - MQTT_SERVER=${MQTT_SERVER}
        depends_on:
        iot-processor:
            condition: service_started
            restart: true
```
<!-- zookeeper kafka kafka-rest-proxy kafka-connect mosquitto mongo grafana prometheus iot-processor iot_sensor_1 -->
<!-- > อธิบายว่า  หน้าจอที่ 1 ที่ต้องเปิดใช้งาน มีการเปิด service อะไรบ้างใน docker -->
## start-service #0
```bash
sh start_0zookeeper_kafka.sh
```
#### service
- zookeeper
- kafka


เมื่อรันคำสั่ง รอจนกว่า service kafka zookeeper จะนิ่งแล้วถึงจะรัน `start-service #1` ต่อไป


## start-service #1
```bash
sh start_1kafka_service.sh
```
#### service
- kafka-rest-proxy
- kafka-connect
- mosquitto
- mongo
- grafana
- prometheus


เมื่อรันคำสั่ง รอจนกว่า terminal จะขึ้นว่า `kafka-connect Kafka Connect listener HTTP state:  000  (waiting for 200)` ถึงจะรัน `start-service #2` ต่อไปได้

## start-service #2
```bash
sh start_2iot_processor.sh
```
#### service
- iot-processor

รอจนกว่า iot-processor จะขึ้นว่า initialize ถึงจะรัน `start-service #3` ต่อไป

## start-service #3
```bash
sh start_3iot_sensor.sh
```
#### service
- iot_sensor



