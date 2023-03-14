# Kafka Deployment

To deploy a Kafka service, two applications need to be up and running. 
1. Zookeeper
2. Kafka
3. (Optional) Kowl


```yaml
version: "3.8"

services:
  zoo1:
    image: zookeeper:3.4.9
    hostname: zoo1
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
    ports:
      - "2181:2181"
    environment:
      ZOO_MY_ID: 1
      ZOO_PORT: 2181
      ZOO_SERVERS: server.1=zoo1:2888:3888

  kafka:
    image: confluentinc/cp-kafka:5.5.1
    hostname: kafka1
    restart: on-failure
    ports:
      - "9092:9092"
      - "29092:29092"
    environment:
      KAFKA_ADVERTISED_LISTENERS: LISTENER_DOCKER_INTERNAL://kafka:29092,LISTENER_DOCKER_EXTERNAL://127.0.0.1:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: LISTENER_DOCKER_INTERNAL:PLAINTEXT,LISTENER_DOCKER_EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: LISTENER_DOCKER_INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181"
      KAFKA_CREATE_TOPICS: notification:1:1:compact
      KAFKA_BROKER_ID: 1
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    depends_on:
      - zoo1
    ulimits:
      nofile:
        soft: 65536
        hard: 65536

  kowl:
    image: quay.io/cloudhut/kowl:master
    ports:
      - "8080:8080"
    environment:
      - KAFKA_BROKERS=kafka:29092
    depends_on:
      - kafka
```
