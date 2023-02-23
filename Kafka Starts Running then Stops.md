
## Problem: 
1. Ran a Docker instance by doing a docker-compose on my .yml file. with the configuration looking like this:
```
broker:
    image: confluentinc/cp-server:7.3.1
    hostname: broker
    container_name: broker
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
      
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://broker:29092
      KAFKA_METRIC_REPORTERS: io.confluent.metrics.reporter.ConfluentMetricsReporter
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_CONFLUENT_LICENSE_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_CONFLUENT_BALANCER_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_JMX_PORT: 9101
      KAFKA_JMX_HOSTNAME: localhost
      KAFKA_CONFLUENT_SCHEMA_REGISTRY_URL: http://schema-registry:8081
      CONFLUENT_METRICS_REPORTER_BOOTSTRAP_SERVERS: broker:29092
      CONFLUENT_METRICS_REPORTER_TOPIC_REPLICAS: 1 
      CONFLUENT_METRICS_ENABLE: 'true'
      CONFLUENT_SUPPORT_CUSTOMER_ID: 'anonymous'
```

The config above worked for weeks and one day, it stopped after i did a docker compose up using it from a .yml file.

## My solution:
1. I realized that my environment settings for Kafka were defective, and preventing Kafka to use Zookeeper.
2. So I found more appropriate settings from https://docs.confluent.io/platform/current/installation/docker/config-reference.html#optional-confluent-enterprise-ak-settings
3. The new settings made me change KAFKA_ZOOKEEPER_CONNECT from "zookeeper:2181" to "localhost:2181"
4. I also changed KAFKA_BROKER_ID to 1, instead of 2.
5. I also changed KAFKA_ADVERTISED_LISTENERS to only PLAINTEXT://localhost:29092 . I eliminated the others

See the code below that stopped my initial config from failing to run after being initialized:

```
docker run -d \
    --net=host \
    --name=kafka \
    -e KAFKA_ZOOKEEPER_CONNECT=localhost:2181 \
    -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:29092 \
    -e KAFKA_BROKER_ID=1 \
    -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 \
    confluentinc/cp-kafka:7.3.1
```


## Result
After running the newly edited code above, problem solved and kafka has run for more than 20 minutes without stopping suddenly anymore!

SCREENSHOT BELOW:
![image](https://user-images.githubusercontent.com/95084188/220904597-aba8d18d-468f-4a44-afc8-c7510063bd3a.png)
