.. DA-ITSI-TELEGRAF-KAFJA documentation master file, created by
   sphinx-quickstart on Tue Sep 18 23:25:46 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to the unified guide for Kafka and Confluent monitoring with Splunk
===========================================================================

**The unified guide for Kafka and Confluent monitoring with Splunk provides a full step by step guidance for monitoring with Splunk, with the following main concepts:**

* realtime event logging
* realtime and high performance metric store
* evolutive and efficient alerting
* scalability and resiliency
* compatibility with traditional bare metails/virtual machines deployments and Kubernetes containers deployments

**As a basis, the following components are being natively managed:**

* Zookeeper
* Apache Kafka Brokers
* Apache Kafka Connect
* Confluent schema-registry
* Confluent ksql-server
* Confluent kafka-rest
* Kafka SLA and end to end monitoring with the Linkedin Kafka monitor

**The following components are leveraged:**

* Splunk (!)
* Jolokia, connector interface for JMX
* Telegraf, the plugin-driven server agent for collecting & reporting metrics

*metrics collection diagram example*

.. image:: img/draw.io/overview_diagram.png
   :alt: overview_diagram
   :align: center

*Kubernetes metrics collection diagram - sidecar containers:*

.. image:: img/draw.io/Kubernetes-deployment.png
   :alt: Kubernetes-deployment.png
   :align: center

Implementation:
===============

.. toctree::
   :maxdepth: 2

   pre-requisites
   chapter1_events
   chapter2_metrics
   chapter3_alerting

