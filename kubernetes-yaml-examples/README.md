# Kubernetes configuration examples

## This repository provides yaml configuration and instructions to monitor Kafka and Confluent infrastructure in Kubernetes

*metrics collection diagram - sidecar containers metrics collection by Telegraf to Jolokia:*

![screen1](../docs/img/draw.io/k8s-metrics.png)

*events logging ingestion diagram - sidecar containers Splunk Universal Forwarders reading logs in pod shared volumes:*

![screen1](../docs/img/draw.io/k8s-logging.png)

--------------------------------------------------------------------------------

### Jolokia deployment

See:

https://splunk-guide-for-kafka-monitoring.readthedocs.io/en/latest/chapter2_metrics.html#id1

yaml samples and quick instructions:

[./Jolokia](./Jolokia/)

--------------------------------------------------------------------------------

### Zookeeper monitoring

yaml samples and quick instructions:

[./zookeeper](./zookeeper/)

--------------------------------------------------------------------------------

### Kafka Brokers monitoring

yaml samples and quick instructions:

[./kafka-brokers](./kafka-brokers/)

--------------------------------------------------------------------------------

### Kafka Connect

yaml samples and quick instructions:

[./kafka-connect](./kafka-connect/)

--------------------------------------------------------------------------------

### Confluent schema-registry

yaml samples and quick instructions:

[./confluent-schema-registry](./confluent-schema-registry/)

--------------------------------------------------------------------------------

### Confluent kafka-rest

yaml samples and quick instructions:

[./confluent-kafka-rest](./confluent-kafka-rest/)

--------------------------------------------------------------------------------

### Confluent ksql-server

yaml samples and quick instructions:

[./confluent-ksql-server](./confluent-ksql-server/)

--------------------------------------------------------------------------------

### Kafka monitor by LinkedIn

yaml samples and quick instructions:

[./LinkedIn-kafka-monitor](./LinkedIn-kafka-monitor/)

--------------------------------------------------------------------------------
[Go back](https://github.com/guilhemmarchand/splunk-guide-for-kafka-monitoring/)