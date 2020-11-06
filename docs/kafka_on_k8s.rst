Running Kafka in Kubernetes
###########################

**Deploying and running successfully Kafka in Kubernetes is not the purpose of the guide, however it is interesting to share some tips about this task which can be quite complex.**

Confluent platform with helm
****************************

**Confluent provides a very interesting set of configurations with helm that you can use to setup and build your infrastructure in Kubernetes:**

* https://docs.confluent.io/current/installation/installing_cp/cp-helm-charts/docs/index.html
* https://github.com/confluentinc/cp-helm-charts

The templates provides in this guide are built on top of the Confluent platform and these configurations, which however can be adapted to run on any kind of deployment.

Testing with minikube
=====================

**Make sure you start minikube with enough memory and cpu resources, example:**

::

    minikube start --memory 8096 --cpus 4

**Before starting the deployment with helm, you can use the following configuration to create the require storage classes:**

*minikube_storageclasses.yml*

::

    ---
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: kafka-broker
    provisioner: k8s.io/minikube-hostpath
    reclaimPolicy: Retain
    ---
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: kafka-zookeeper
    provisioner: k8s.io/minikube-hostpath
    reclaimPolicy: Retain

**Then apply:**

::

    kubectl create -f minikube_storageclasses.yml

**Modify the values.yml to include the storage classes and some restrictions on the containers to get it successful:**

*values.yaml*

::

    ## ------------------------------------------------------
    ## Zookeeper
    ## ------------------------------------------------------
    cp-zookeeper:
      enabled: true
      servers: 3
      image: confluentinc/cp-zookeeper
      imageTag: 5.0.1
      ## Optionally specify an array of imagePullSecrets. Secrets must be manually created in the namespace.
      ## https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod
      imagePullSecrets:
      #  - name: "regcred"
      heapOptions: "-Xms512M -Xmx512M"
      persistence:
        enabled: true
        ## The size of the PersistentVolume to allocate to each Zookeeper Pod in the StatefulSet. For
        ## production servers this number should likely be much larger.
        ##
        ## Size for Data dir, where ZooKeeper will store the in-memory database snapshots.
        dataDirSize: 5Gi
        dataDirStorageClass: "kafka-zookeeper"

        ## Size for data log dir, which is a dedicated log device to be used, and helps avoid competition between logging and snaphots.
        dataLogDirSize: 5Gi
        dataLogDirStorageClass: "kafka-zookeeper"
      resources:
      ## If you do want to specify resources, uncomment the following lines, adjust them as necessary,
      ## and remove the curly braces after 'resources:'
        limits:
         cpu: 100m
         memory: 256Mi
        requests:
         cpu: 100m
         memory: 256Mi

    ## ------------------------------------------------------
    ## Kafka
    ## ------------------------------------------------------
    cp-kafka:
      enabled: true
      brokers: 3
      image: confluentinc/cp-kafka
      imageTag: 5.0.1
      ## Optionally specify an array of imagePullSecrets. Secrets must be manually created in the namespace.
      ## https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod
      imagePullSecrets:
      #  - name: "regcred"
      heapOptions: "-Xms512M -Xmx512M"
      persistence:
        enabled: true
        storageClass: "kafka-broker"
        size: 5Gi
        disksPerBroker: 1
      resources:
      ## If you do want to specify resources, uncomment the following lines, adjust them as necessary,
      ## and remove the curly braces after 'resources:'
        limits:
         cpu: 200m
         memory: 512Mi
        requests:
         cpu: 200m
         memory: 512Mi

    ## ------------------------------------------------------
    ## Schema Registry
    ## ------------------------------------------------------
    cp-schema-registry:
      enabled: true
      image: confluentinc/cp-schema-registry
      imageTag: 5.0.1
      ## Optionally specify an array of imagePullSecrets. Secrets must be manually created in the namespace.
      ## https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod
      imagePullSecrets:
      #  - name: "regcred"
      heapOptions: "-Xms512M -Xmx512M"
      resources:
      ## If you do want to specify resources, uncomment the following lines, adjust them as necessary,
      ## and remove the curly braces after 'resources:'
        limits:
         cpu: 100m
         memory: 512Mi
        requests:
         cpu: 100m
         memory: 512Mi

    ## ------------------------------------------------------
    ## REST Proxy
    ## ------------------------------------------------------
    cp-kafka-rest:
      enabled: true
      image: confluentinc/cp-kafka-rest
      imageTag: 5.0.1
      ## Optionally specify an array of imagePullSecrets. Secrets must be manually created in the namespace.
      ## https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod
      imagePullSecrets:
      #  - name: "regcred"
      resources:
      ## If you do want to specify resources, uncomment the following lines, adjust them as necessary,
      ## and remove the curly braces after 'resources:'
        limits:
         cpu: 100m
         memory: 256Mi
        requests:
         cpu: 100m
         memory: 256Mi

    ## ------------------------------------------------------
    ## Kafka Connect
    ## ------------------------------------------------------
    cp-kafka-connect:
      enabled: true
      image: confluentinc/cp-kafka-connect
      imageTag: 5.0.1
      ## Optionally specify an array of imagePullSecrets. Secrets must be manually created in the namespace.
      ## https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod
      imagePullSecrets:
      #  - name: "regcred"
      resources: {}
      ## If you do want to specify resources, uncomment the following lines, adjust them as necessary,
      ## and remove the curly braces after 'resources:'
        #limits:
         #cpu: 100m
         #memory: 512Mi
        #requests:
         #cpu: 100m
         #memory: 512Mi

    ## ------------------------------------------------------
    ## KSQL Server
    ## ------------------------------------------------------
    cp-ksql-server:
      enabled: true
      image: confluentinc/cp-ksql-server
      imageTag: 5.0.1
      ## Optionally specify an array of imagePullSecrets. Secrets must be manually created in the namespace.
      ## https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod
      imagePullSecrets:
      #  - name: "regcred"
      ksql:
        headless: false

**Starting helm:**

The templates provided are built on the naming convention of a helm installion called "confluent-oss" in a name space called "kafka":

*helm3 command:*

::

    helm install confluent-oss confluentinc/cp-helm-charts

The following command is valid for helm2 only, and left for historical purposes

::

    helm install cp-helm-charts --name confluent-oss --namespace kafka

**The helm installation provided by Confluent will create:**

- Zookeeper cluster in a statefulSet
- Kafka Brokers cluster in a statefulSet
- Kafka Connect in a Deployment
- Confluent Schema registry in a Deployment
- Confluent ksql-server in a Deployment
- Confluent kafka-rest in a Deployment
