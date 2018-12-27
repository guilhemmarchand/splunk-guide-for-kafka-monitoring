# Splunk Universal Forwarder sidecar container to monitor events logging of Kafka brokers in a Kubernetes statefulSet

**See:**

--------------------------------------------------------------------------------

This configuration guide assumes that you have a Splunk Deployment Server (DS) that the Splunk Universal Forwarders sidecar containers will contact to retrieve specific configuration items.

Specially, your Deployment Server needs to deploy automatically the outputs configuration that provides the definition for your indexing layer. (outputs.conf)

If you do not want to use a Deployment Server for this purpose, you can modify the following configuration and provides the outputs definition in a configMap. (see 02-kafka-brokers-splunk-inputs.yml/05-patch-shared-volume-and-splunk-uf.yml)

### Step 1: (Splunk secrets and configMap)

- Ensure you have created a secrets to reference the Splunk admin password:

*Convert the Splunk admin password in base64 value:*

```
echo "ch@ngeM3" | base64
Y2hAbmdlTTMK
```

*Update your secrets yaml file: ../../yaml_git_ignored/global-splunk-uf-secrets.yml:*

```
apiVersion: v1
kind: Secret
metadata:
  name: splunk-secrets
  namespace: kafka
type: Opaque
data:
  splunk_password: Y2hAbmdlTTMK
```

*Create:*

```
kubectl create -f ../../yaml_git_ignored/global-splunk-uf-secrets.yml
```

- Ensure you have created a configMap to reference the Splunk deployment server URL:

*../yaml_git_ignored/global-splunk-uf-config.yml:*

```
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: kafka
  name: global-splunk-uf-config
data:
  splunk_deployment_server: "my-splunk-ds-url.amazonaws.com"
  splunk_s2s_port: "8089"
```

*Create:*

```
kubectl create -f ../../yaml_git_ignored/global-splunk-uf-config.yml
```

--------------------------------------------------------------------------------

### Step 2: (Splunk inputs configMap)

This configuration create the inputs for the Splunk Universal Forwarder to monitor and index the Kafka brokers events.

The default index is named "kafka", if you wish to change this, update the configMap before its creation.

Make sure the index has been created in your Splunk indexing layer.

*Create:*

```
kubectl create -f 02-kafka-brokers-splunk-inputs.yml
```

### Step 3: (KAFKA_OPTS configMap)

*Create:*

```
kubectl create -f 03-kafka-brokers-opts-configmap.yml
```

--------------------------------------------------------------------------------

### Step 4: (log4j configMap)

*Create:*

```
kubectl create -f 04-kafka-brokers-log4j-configmap.yml

```

--------------------------------------------------------------------------------

### Step 5: (patch)

The patch will update your Kafka broker statefulSet deployment and create the Splunk Universal Forwarder sidecar container.

- Update the file 05-patch-shared-volume-and-splunk-uf.yml to match the name of your statefulSet deployment

*This part must be changed to match the name of your statefulSet deployment:*

```
metadata:
  name: confluent-oss-cp-kafka
```

- Run the patch command and ensure you specify the name of your statefulSet deployment:

```
kubectl --namespace kafka patch statefulset confluent-oss-cp-kafka --patch "$(cat 05-patch-shared-volume-and-splunk-uf.yml )"
```

--------------------------------------------------------------------------------

**To troubleshoot, useful kubectl commands:**

```
kubectl -n kafka describe statefulSet.apps confluent-oss-cp-kafka
kubectl -n kafka get po
kubectl -n kafka describe po confluent-oss-cp-kafka-0
kubectl -n kafka logs confluent-oss-cp-kafka-0 -c splunk
kubectl -n kafka logs confluent-oss-cp-kafka-0 -c cp-kafka-broker
```

--------------
[Go back](../)