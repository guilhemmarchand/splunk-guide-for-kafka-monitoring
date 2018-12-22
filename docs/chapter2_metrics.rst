Chapter 2: metrics
##################

Monitoring Kafka in dedicated servers (bare metal, VMs)
*******************************************************

.. image:: img/dedicated-server.png
   :alt: dedicated-server.png
   :align: center

**Dedicated servers are bare metal servers or virtual machines that are dedicated to host one or more Kafka roles.**

Deploying Jolokia
=================

.. image:: img/jolokia_logo.png
   :alt: jolokia_logo.png
   :align: center

**The first requirement is the deployment of Jolokia to your Kafka components:**

* Apache Kafka Brokers
* Apache Kafka Connect
* Confluent schema-registry
* Confluent ksql-server
* Confluent kafka-rest

*Notes: Jolokia does not need to be deployed to your Zookeeper nodes as we use the Telegraph Zookeeper input.*

Jolokia is a very simple and efficient JVM agent to deployed to the components, it can be started by attaching to the JVM process on the flight, or automatically at JVM startup by using the javaagent argument.

For more information about Jolokia: https://jolokia.org

- Download the latest Jolokia JVM agent jar file: https://jolokia.org/reference/html/agents.html#agents-jvm

- Upload the jar file to each of your servers, this documentation assumes the agent will be available in ``/opt/jolokia/``, example:

::

    /opt/jolokia/jolokia-jvm-1.6.0-agent.jar

- Modify the systemd service file to configure the ``-javaagent`` argument.

*Notes:*

- Choose one TCP port per Jolokia instance
- Choose if you want to allow Jolokia to listen on any interfaces (``host=0.0.0.0``) or restricted to a specific interface (depending how you collect, see later on the documentation)
- More options are available, see https://jolokia.org/reference/html/agents.html

Kafka brokers
=============

**Configuring the systemd service file:**

- Edit: ``/lib/systemd/system/confluent-kafka.service``

- Add ``-javaagent`` argument:

::

    [Unit]
    Description=Apache Kafka - broker
    Documentation=http://docs.confluent.io/
    After=network.target confluent-zookeeper.target

    [Service]
    Type=simple
    User=cp-kafka
    Group=confluent
    ExecStart=/usr/bin/kafka-server-start /etc/kafka/server.properties
    Environment="KAFKA_OPTS=-javaagent:/opt/jolokia/jolokia.jar=port=8778,host=0.0.0.0"
    TimeoutStopSec=180
    Restart=no

    [Install]
    WantedBy=multi-user.target

- Reload systemd and restart:

::

    sudo systemctl daemon-restart
    sudo systemctl restart confluent-kafka

Kafka Connect
=============

**Configuring the systemd service file:**

- Edit: ``/lib/systemd/system/confluent-kafka-connect.service``

- Add ``-javaagent`` argument:

::

    [Unit]
    Description=Apache Kafka Connect - distributed
    Documentation=http://docs.confluent.io/
    After=network.target confluent-kafka.target

    [Service]
    Type=simple
    User=cp-kafka-connect
    Group=confluent
    ExecStart=/usr/bin/connect-distributed /etc/kafka/connect-distributed.properties
    Environment="KAFKA_OPTS=-javaagent:/opt/jolokia/jolokia.jar=port=8778,host=0.0.0.0"
    Environment="LOG_DIR=/var/log/connect"
    TimeoutStopSec=180
    Restart=no

    [Install]
    WantedBy=multi-user.target

- Reload systemd and restart:

::

    sudo systemctl daemon-restart
    sudo systemctl restart confluent-kafka-connect

ksql-server
===========

**Configuring the systemd service file:**

- Edit: ``/lib/systemd/system/confluent-ksql.service``

- Add ``-javaagent`` argument:

::

    [Unit]
    Description=Streaming SQL engine for Apache Kafka
    Documentation=http://docs.confluent.io/
    After=network.target confluent-kafka.target confluent-schema-registry.target

    [Service]
    Type=simple
    User=cp-ksql
    Group=confluent
    Environment="LOG_DIR=/var/log/confluent/ksql"
    Environment="KSQL_OPTS=-javaagent:/opt/jolokia/jolokia.jar=port=8778,host=0.0.0.0"
    ExecStart=/usr/bin/ksql-server-start /etc/ksql/ksql-server.properties
    TimeoutStopSec=180
    Restart=no

    [Install]
    WantedBy=multi-user.target

- Reload systemd and restart:

::

    sudo systemctl daemon-restart
    sudo systemctl restart confluent-ksql


kafka-rest
==========

**Configuring the systemd service file:**

- Edit: ``/lib/systemd/system/confluent-kafka-rest.service``

- Add ``-javaagent`` argument:

::

    [Unit]
    Description=A REST proxy for Apache Kafka
    Documentation=http://docs.confluent.io/
    After=network.target confluent-kafka.target

    [Service]
    Type=simple
    User=cp-kafka-rest
    Group=confluent
    Environment="LOG_DIR=/var/log/confluent/kafka-rest"
    Environment="KAFKAREST_OPTS=-javaagent:/opt/jolokia/jolokia.jar=port=8778,host=0.0.0.0"


    ExecStart=/usr/bin/kafka-rest-start /etc/kafka-rest/kafka-rest.properties
    TimeoutStopSec=180
    Restart=no

    [Install]
    WantedBy=multi-user.target

- Reload systemd and restart:

::

    sudo systemctl daemon-restart
    sudo systemctl restart confluent-kafka-rest

Monitoring Kafka in Kubernetes
******************************

.. image:: img/kubernetes-logo.png
   :alt: kubernetes-logo.png
   :align: center

**For the ease of documentation, this guide assumes you are deploying containers with Kubernetes and Docker, although these instructions can transposed to other containers orchestrator solutions.**

**3 main steps for implementation:**

1. Deploying Jolokia jar agent
2. Configuring the containers to start with Jolokia
3. Deploying the Telegraf containers

Deploying Jolokia
=================

.. image:: img/jolokia_logo.png
   :alt: jolokia_logo.png
   :align: center

**The Jolokia agent jar file needs to be available to the pods, you have different possibilities:**

- Starting Kubernetes 1.10.0, you can store a binary file in a configMap. As such, it is very easy to load the Jolokia jar file and make it available to your pods. (**recommended approach**)

- For prior versions, you can automatically mount a persistent volume on the pods such as an NFS volume or a Cloud provider volume that will make the Jolokia jar available to your pods.

- uploading the jar file on every node and mounting a local persistent volume (requires each node to have the jolokia jar uploaded manually)

**To download the latest version of Jolokia:** https://jolokia.org/reference/html/agents.html#agents-jvm

Option 1: Jolokia jar in configMap
----------------------------------

**See the files in Github:**

https://github.com/guilhemmarchand/splunk-guide-for-kafka-monitoring/tree/master/kubernetes-yaml-examples/Jolokia

**From your management server where kubectl is configured, download the latest Jolokia jar file:**

::

    curl http://search.maven.org/remotecontent?filepath=org/jolokia/jolokia-jvm/1.6.0/jolokia-jvm-1.6.0-agent.jar -o jolokia.jar

**Create a configMap from the binary file:**

::

    kubectl create configmap jolokia-jar --from-file=jolokia.jar

**From the configMap, optionally create the yml file:**

::

    kubectl get configmaps jolokia-jar -o yaml --export > jolokia-jar-configMap.yml

**If you need your configMap to be associated with a name space, simply edit the end of the file and add your name space Metadata:**

::

    metadata:
      name: jolokia-jar
      namespace: kafka

**Modify your definitions to include the volume:**

::

    spec:
      volumes:
        - name: jolokia-jar
          configMap:
            name: jolokia-jar
      containers:
        - name: xxxxx
          image: xxxx
          volumeMounts:
            - mountPath: "/opt/jolokia"
              name: jolokia-jar

**Finally, update the environment variable to start Jolokia (see next steps) and apply.**

Option 2: NFS persistent volume configuration example
-----------------------------------------------------

**Ensure all the nodes have the nfs-common package installed:**

*For Ubuntu & Debian:*

::

    sudo apt-get -y install nfs-common

*For RHEL, Centos and derivated:*

::

    sudo yum -y install nfs-common

**Upload the jar file to your NFS server, and create a share that will be used automatically by the pods, example:**

::

    /export/jolokia/jolokia-jvm-1.6.0-agent.jar

**Have your share configured in /etc/exports:**

::

    /export/jolokia/ *(ro,sync,no_root_squash,subtree_check)

**Refresh exports:**

::

    sudo exportfs -ra

**Create a Kubernetes PersistentVolume:**

*pv-jolokia.yaml*

::

    kind: PersistentVolume
    apiVersion: v1
    metadata:
      name: pv-jolokia
      labels:
        type: jolokia
    spec:
      storageClassName: generic
      capacity:
        storage: 100Mi
      accessModes:
        - ReadOnlyMany
      persistentVolumeReclaimPolicy: Retain
      nfs:
        path: /export/jolokia
        server: <NFS server address>
        readOnly: true

*pvc-jolokia.yaml:**

::

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: pvc-jolokia
    spec:
      storageClassName: generic
      accessModes:
      - ReadOnlyMany
      resources:
        requests:
          storage: 100Mi
      selector:
        matchLabels:
          type: jolokia

**When you will start your pods, you will specify the PersistentVolumeClaim and the mount options to get Jolokia available on the pods:**

::

    kind: Pod
    apiVersion: v1
    metadata:
      name: xxxxx
    spec:
      volumes:
        - name: pv-jolokia
          persistentVolumeClaim:
           claimName: pvc-jolokia
      containers:
        - name: xxxxx
          image: xxxx
          volumeMounts:
            - mountPath: "/opt/jolokia"
              name: pv-jolokia

Option 3: Local persistent volume configuration example
-------------------------------------------------------

**Upload the jar file to each of Kubernetes node, this documentation assumes the agent will be available in /opt/jolokia/, example:**

::

    /opt/jolokia/jolokia-jvm-1.6.0-agent.jar

**Create a Kubernetes PersistentVolume:**

*pv-jolokia.yaml*

::

    kind: PersistentVolume
    apiVersion: v1
    metadata:
      name: pv-jolokia
      labels:
        type: jolokia
    spec:
      storageClassName: generic
      capacity:
        storage: 100Mi
      accessModes:
        - ReadOnlyMany
      persistentVolumeReclaimPolicy: Retain
      hostPath:
        path: "/opt/jolokia"

**Create:**

::

    kubectl create -f pv-jolokia.yaml

**Create a PersistentVolumeClaim to be used by the pods definition:**

*pvc-jolokia.yaml:**

::

        apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: pvc-jolokia
        spec:
          storageClassName: generic
          accessModes:
          - ReadOnlyMany
          resources:
            requests:
              storage: 100Mi
          selector:
            matchLabels:
              type: jolokia

**When you will start your pods, you will specify the PersistentVolumeClaim and the mount options to get Jolokia available on the pods:**

::

    kind: Pod
    apiVersion: v1
    metadata:
      name: xxxxx
    spec:
      volumes:
        - name: pv-jolokia
          persistentVolumeClaim:
           claimName: pvc-jolokia
      containers:
        - name: xxxxx
          image: xxxx
          volumeMounts:
            - mountPath: "/opt/jolokia"
              name: pv-jolokia

Starting Jolokia with container startup
=======================================

Kafka brokers
-------------

**Modify your pod definition:**

::

    spec:
      containers:
      - name: xxxxxx
        image: xxxxxx:latest
        env:
        - name: KAFKA_OPTS
          value: "-javaagent:/opt/jolokia/jolokia.jar=port=8778,host=0.0.0.0"

Kafka Connect
-------------

**Modify your pod definition:**

::

    spec:
      containers:
      - name: xxxxxx
        image: xxxxxx:latest
        env:
        - name: KAFKA_OPTS
          value: "-javaagent:/opt/jolokia/jolokia.jar=port=8778,host=0.0.0.0"

Schema registry
---------------

**Modify your pod definition:**

::

    spec:
      containers:
      - name: xxxxxx
        image: xxxxxx:latest
        env:
        - name: SCHEMA_REGISTRY_OPTS
          value: "-javaagent:/opt/jolokia/jolokia.jar=port=8778,host=0.0.0.0"

ksql-server
-----------

**Modify your pod definition:**

::

    spec:
      containers:
      - name: xxxxxx
        image: xxxxxx:latest
        env:
        - name: KSQL_OPTS
          value: "-javaagent:/opt/jolokia/jolokia.jar=port=8778,host=0.0.0.0"

kafka-rest
----------

**Modify your pod definition:**

::

    spec:
      containers:
      - name: xxxxxx
        image: xxxxxx:latest
        env:
        - name: KAFKAREST_OPTS
          value: "-javaagent:/opt/jolokia/jolokia.jar=port=8778,host=0.0.0.0"

Deploying Telegraf
==================

.. image:: img/telegraf-logo.png
   :alt: telegraf-logo.png
   :align: center

**Telegraf is a very efficient plugin driven agent collector, in the context of Kubernetes there are several design choices possible:**

- Running Telegraf agent as a container in the same pod than the JVM container, called a sidecar container. (recommended approach)
- Running Telegraf agent as a deployment with 1 replica, accessing all JVMs instances via cluster exposed services (one or more deployments if you want to specialise per role, or something else)

Both designs are pertinents, however running collector agents as sidecar containers provides valuable advantages such as ensuring that the collector container will always run on the same node.

In addition, this is an easy "build and forget" approach, each container monitors the local JVM container automatically, following the same rhythm of destruction and creation.

Option 1: Telegraf as a sidecar container
-----------------------------------------

**When running Telegraf as a sidecar container, an additional container will be running in the same pod, generally associated with a StatefulSet.**

*The following yaml example defines the configMap containing the telegraf.conf configuration for a Kafka broker:*

*Notes:*

- The url targeting the Splunk HEC and the token values need to be updated according to your environment
- verify and modify namespace
- observe the usage of a variable "$POD_NAME" in the Jolokia URL, this is required to be able to identify properly the instance

*telegraf-config-kafka-broker.yml*

::

    kind: ConfigMap
    metadata:
      name: telegraf-config-kafka-broker
      namespace: kafka
    apiVersion: v1
    data:

      telegraf.conf: |+
        [global_tags]
          env = "$ENV"
        [agent]
          hostname = "$HOSTNAME"
        [[outputs.http]]
          ## URL is the address to send metrics to
          url = "https://splunk.mydomain.com:8088/services/collector"
          ## Timeout for HTTP message
          # timeout = "5s"
          ## Optional TLS Config
          # tls_ca = "/etc/telegraf/ca.pem"
          # tls_cert = "/etc/telegraf/cert.pem"
          # tls_key = "/etc/telegraf/key.pem"
          ## Use TLS but skip chain & host verification
          insecure_skip_verify = true
          ## Data format to output.
          ## Each data format has it's own unique set of configuration options, read
          ## more about them here:
          ## https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_OUTPUT.md
          data_format = "splunkmetric"
          ## Provides time, index, source overrides for the HEC
          splunkmetric_hec_routing = true
          ## Additional HTTP headers
          [outputs.http.headers]
            # Should be set manually to "application/json" for json data_format
            Content-Type = "application/json"
            #Authorization = "Splunk 65735c4b-f277-4f69-87ca-ff2b738c69f9"
            #X-Splunk-Request-Channel = "65735c4b-f277-4f69-87ca-ff2b738c69f9"
            Authorization = "Splunk 205d43f1-2a31-4e60-a8b3-327eda49944a"
            X-Splunk-Request-Channel = "205d43f1-2a31-4e60-a8b3-327eda49944a"

        # Kafka JVM monitoring

        [[inputs.jolokia2_agent]]
          name_prefix = "kafka_"
          urls = ["http://$POD_NAME:8778/jolokia"]

        [[inputs.jolokia2_agent.metric]]
          name         = "controller"
          mbean        = "kafka.controller:name=*,type=*"
          field_prefix = "$1."

        [[inputs.jolokia2_agent.metric]]
          name         = "replica_manager"
          mbean        = "kafka.server:name=*,type=ReplicaManager"
          field_prefix = "$1."

        [[inputs.jolokia2_agent.metric]]
          name         = "purgatory"
          mbean        = "kafka.server:delayedOperation=*,name=*,type=DelayedOperationPurgatory"
          field_prefix = "$1."
          field_name   = "$2"

        [[inputs.jolokia2_agent.metric]]
          name     = "client"
          mbean    = "kafka.server:client-id=*,type=*"
          tag_keys = ["client-id", "type"]

        [[inputs.jolokia2_agent.metric]]
          name         = "network"
          mbean        = "kafka.network:name=*,request=*,type=RequestMetrics"
          field_prefix = "$1."
          tag_keys     = ["request"]

        [[inputs.jolokia2_agent.metric]]
          name         = "network"
          mbean        = "kafka.network:name=ResponseQueueSize,type=RequestChannel"
          field_prefix = "ResponseQueueSize"
          tag_keys     = ["name"]

        [[inputs.jolokia2_agent.metric]]
          name         = "network"
          mbean        = "kafka.network:name=NetworkProcessorAvgIdlePercent,type=SocketServer"
          field_prefix = "NetworkProcessorAvgIdlePercent"
          tag_keys     = ["name"]

        [[inputs.jolokia2_agent.metric]]
          name         = "topics"
          mbean        = "kafka.server:name=*,type=BrokerTopicMetrics"
          field_prefix = "$1."

        [[inputs.jolokia2_agent.metric]]
          name         = "topic"
          mbean        = "kafka.server:name=*,topic=*,type=BrokerTopicMetrics"
          field_prefix = "$1."
          tag_keys     = ["topic"]

        [[inputs.jolokia2_agent.metric]]
          name       = "partition"
          mbean      = "kafka.log:name=*,partition=*,topic=*,type=Log"
          field_name = "$1"
          tag_keys   = ["topic", "partition"]

        [[inputs.jolokia2_agent.metric]]
          name       = "log"
          mbean      = "kafka.log:name=LogFlushRateAndTimeMs,type=LogFlushStats"
          field_name = "LogFlushRateAndTimeMs"
          tag_keys   = ["name"]

        [[inputs.jolokia2_agent.metric]]
          name       = "partition"
          mbean      = "kafka.cluster:name=UnderReplicated,partition=*,topic=*,type=Partition"
          field_name = "UnderReplicatedPartitions"
          tag_keys   = ["topic", "partition"]

        [[inputs.jolokia2_agent.metric]]
          name     = "request_handlers"
          mbean    = "kafka.server:name=RequestHandlerAvgIdlePercent,type=KafkaRequestHandlerPool"
          tag_keys = ["name"]

        # JVM garbage collector monitoring
        [[inputs.jolokia2_agent.metric]]
          name     = "jvm_garbage_collector"
          mbean    = "java.lang:name=*,type=GarbageCollector"
          paths    = ["CollectionTime", "CollectionCount", "LastGcInfo"]
          tag_keys = ["name"]

**Apply:**

::

    kubectl create -f telegraf-config-kafka-broker.yml

*The following yaml create the additional container within the StatefulSet:*

*Notes:*

- The "name: kafka" in the example bellow matches the name of the StatefulSet's pods
- The namespace needs to be modified depending on the configuration
- The "POD_NAME" variable used in the Jolokia URL is automatically defined from Kubernetes metadata

*telegraf-kafka-broker.yml*

::

    # meant to be applied using
    # kubectl --namespace kafka patch statefulset kafka --patch "$(cat filename.yml )"
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: kafka
      namespace: kafka
    spec:
      template:
        spec:
          containers:
          - name: telegraf
            image: docker.io/telegraf:latest
            resources:
              limits:
                memory: 500Mi
              requests:
                cpu: 100m
                memory: 500Mi
            env:
            - name: HOSTNAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            volumeMounts:
            - name: telegraf-config-kafka-broker
              mountPath: /etc/telegraf
          volumes:
          - name: telegraf-config-kafka-broker
            configMap:
              name: telegraf-config-kafka-broker

**Patch the statefulSet:**

::

    kubectl --namespace kafka patch statefulset kafka --patch "$(cat telegraf-kafka-broker.yml )"

**To see logs from the side card container, example:**

::

    kubectl -n <namespace> logs <pod_name>-<pod_id> -c telegraf

**To open a terminal in the container, example:**

::

    kubectl -n <namespace> exec -it <pod_name>-<pod_id> -c telegraf /bin/bash

Option 2: Telegraf as a deployment
----------------------------------

Running Telegraf as a deployment is basically achieving a remote collection of multiple instances from an independent pod and container, using 1 replica Kubernetes will ensure that always have at least 1 container running.

Expose the Kafka services
^^^^^^^^^^^^^^^^^^^^^^^^^

**The first requirement is having the services exposed, such that the Telegraf container will always be able to reach the Jolokia instances relying on the Kubernetes DNS records and services discovery.**

**Assuming a 3 pods Kafka brokers statefulSet, you can expose the Jolokia services using kubectl:**

*Notes: the services do not need to be accessible from outside of the cluster*

*The bellow exposes an example for Kafka brokers, the same must be achieved for Zookeeper and any other component*

::

    kubectl expose -n kafka pod kafka-0 --name kafka-0-jolokia --port 8778
    kubectl expose -n kafka pod kafka-1 --name kafka-1-jolokia --port 8778
    kubectl expose -n kafka pod kafka-2 --name kafka-2-jolokia --port 8778

**Verify the services:**

::

    kubectl -n kafka get svc

**Example:**

::

    NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
    kafka-0-jolokia   ClusterIP   10.96.191.196    <none>        8778/TCP            1m
    kafka-1-jolokia   ClusterIP   10.108.21.64     <none>        8778/TCP            12s
    kafka-2-jolokia   ClusterIP   10.100.124.188   <none>        8778/TCP            7s

**In the example above, given that the pods are linked to the kafka namespace, the Jolokia url target will be:**

::

    urls = ["http://kafka-0-svc.kafka.svc.cluster.local:8778/jolokia","http://kafka-1-svc.kafka.svc.cluster.local:8778/jolokia","http://kafka-2-svc.kafka.svc.cluster.local:8778/jolokia"]

Create the Telegraf deployment
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**The following example will create a deployment of 1 replica that monitors all the Kafka brokers:**

*telegraf-deployment-kafka-broker.yml*

::

    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: telegraf-kafka-brokers
      namespace: kafka
      labels:
        k8s-app: telegraf-kafka-brokers
    data:
      telegraf.conf: |+
        [global_tags]
          env = "$ENV"
        [agent]
          hostname = "$HOSTNAME"
        [[outputs.http]]
          ## URL is the address to send metrics to
          url = "https://splunk.mydomain.com:8088/services/collector"
          ## Timeout for HTTP message
          # timeout = "5s"
          ## Optional TLS Config
          # tls_ca = "/etc/telegraf/ca.pem"
          # tls_cert = "/etc/telegraf/cert.pem"
          # tls_key = "/etc/telegraf/key.pem"
          ## Use TLS but skip chain & host verification
          insecure_skip_verify = true
          ## Data format to output.
          ## Each data format has it's own unique set of configuration options, read
          ## more about them here:
          ## https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_OUTPUT.md
          data_format = "splunkmetric"
          ## Provides time, index, source overrides for the HEC
          splunkmetric_hec_routing = true
          ## Additional HTTP headers
          [outputs.http.headers]
            # Should be set manually to "application/json" for json data_format
            Content-Type = "application/json"
            #Authorization = "Splunk 65735c4b-f277-4f69-87ca-ff2b738c69f9"
            #X-Splunk-Request-Channel = "65735c4b-f277-4f69-87ca-ff2b738c69f9"
            Authorization = "Splunk 205d43f1-2a31-4e60-a8b3-327eda49944a"
            X-Splunk-Request-Channel = "205d43f1-2a31-4e60-a8b3-327eda49944a"

        #[[inputs.prometheus]]
        #  urls = ["http://kafka-0-svc.kafka.svc.cluster.local:5556/metrics"]

        # Kafka JVM monitoring

        [[inputs.jolokia2_agent]]
          name_prefix = "kafka_"
          urls = ["http://kafka-0-svc.kafka.svc.cluster.local:8778/jolokia","http://kafka-1-svc.kafka.svc.cluster.local:8778/jolokia","http://kafka-2-svc.kafka.svc.cluster.local:8778/jolokia"]

        [[inputs.jolokia2_agent.metric]]
          name         = "controller"
          mbean        = "kafka.controller:name=*,type=*"
          field_prefix = "$1."

        [[inputs.jolokia2_agent.metric]]
          name         = "replica_manager"
          mbean        = "kafka.server:name=*,type=ReplicaManager"
          field_prefix = "$1."

        [[inputs.jolokia2_agent.metric]]
          name         = "purgatory"
          mbean        = "kafka.server:delayedOperation=*,name=*,type=DelayedOperationPurgatory"
          field_prefix = "$1."
          field_name   = "$2"

        [[inputs.jolokia2_agent.metric]]
          name     = "client"
          mbean    = "kafka.server:client-id=*,type=*"
          tag_keys = ["client-id", "type"]

        [[inputs.jolokia2_agent.metric]]
          name         = "network"
          mbean        = "kafka.network:name=*,request=*,type=RequestMetrics"
          field_prefix = "$1."
          tag_keys     = ["request"]

        [[inputs.jolokia2_agent.metric]]
          name         = "network"
          mbean        = "kafka.network:name=ResponseQueueSize,type=RequestChannel"
          field_prefix = "ResponseQueueSize"
          tag_keys     = ["name"]

        [[inputs.jolokia2_agent.metric]]
          name         = "network"
          mbean        = "kafka.network:name=NetworkProcessorAvgIdlePercent,type=SocketServer"
          field_prefix = "NetworkProcessorAvgIdlePercent"
          tag_keys     = ["name"]

        [[inputs.jolokia2_agent.metric]]
          name         = "topics"
          mbean        = "kafka.server:name=*,type=BrokerTopicMetrics"
          field_prefix = "$1."

        [[inputs.jolokia2_agent.metric]]
          name         = "topic"
          mbean        = "kafka.server:name=*,topic=*,type=BrokerTopicMetrics"
          field_prefix = "$1."
          tag_keys     = ["topic"]

        [[inputs.jolokia2_agent.metric]]
          name       = "partition"
          mbean      = "kafka.log:name=*,partition=*,topic=*,type=Log"
          field_name = "$1"
          tag_keys   = ["topic", "partition"]

        [[inputs.jolokia2_agent.metric]]
          name       = "log"
          mbean      = "kafka.log:name=LogFlushRateAndTimeMs,type=LogFlushStats"
          field_name = "LogFlushRateAndTimeMs"
          tag_keys   = ["name"]

        [[inputs.jolokia2_agent.metric]]
          name       = "partition"
          mbean      = "kafka.cluster:name=UnderReplicated,partition=*,topic=*,type=Partition"
          field_name = "UnderReplicatedPartitions"
          tag_keys   = ["topic", "partition"]

        [[inputs.jolokia2_agent.metric]]
          name     = "request_handlers"
          mbean    = "kafka.server:name=RequestHandlerAvgIdlePercent,type=KafkaRequestHandlerPool"
          tag_keys = ["name"]

        # JVM garbage collector monitoring
        [[inputs.jolokia2_agent.metric]]
          name     = "jvm_garbage_collector"
          mbean    = "java.lang:name=*,type=GarbageCollector"
          paths    = ["CollectionTime", "CollectionCount", "LastGcInfo"]
          tag_keys = ["name"]

    ---
    # Section: Deployment
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: telegraf-kafka-brokers
      namespace: kafka
      labels:
        k8s-app: telegraf-kafka-brokers
    spec:
      replicas: 1
      selector:
        matchLabels:
          name: telegraf-kafka-brokers
      template:
        metadata:
          labels:
            name: telegraf-kafka-brokers
        spec:
          containers:
          - name: telegraf
            image: docker.io/telegraf:latest
            resources:
              limits:
                memory: 500Mi
              requests:
                cpu: 100m
                memory: 500Mi
            env:
            - name: HOSTNAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            volumeMounts:
            - name: telegraf-kafka-brokers-config
              mountPath: /etc/telegraf
          terminationGracePeriodSeconds: 30
          volumes:
          - name: config
            configMap:
              name: telegraf-kafka-brokers

**Apply:**

::

    kubectl create -f telegraf-deployment-kafka-broker.yml
