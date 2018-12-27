# Telegraf sidecar container to monitor Kafka Connect in Kubernetes deployment

**See:**

https://splunk-guide-for-kafka-monitoring.readthedocs.io/en/latest/chapter2_metrics.html#option-1-telegraf-as-a-sidecar-container

--------------------------------------------------------------------------------

### Step 1: (Splunk url and token environment variables)

- Ensure you have created a configMap to reference the environment name, Splunk HEC url and token values that will be used by all your pods:

*../../yaml_git_ignored/global-config.yml:*

```
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: kafka
  name: global-config
data:
  env: my-environment
  splunk_hec_url: my-splunk-hec.domain.com:8088
  splunk_hec_token: 205d43f1-2a31-4e60-a8b3-327eda49944a
```

*Create:*

```
kubectl create -f ../../yaml_git_ignored/global-config.yml
```

The "splunk_hec_url" and "splunk_hec_token" are automatically substituted by the according values.

- Create the Kubernetes configMap:

```
kubectl create -f 01-telegraf-config-kafka-connect.yml
```

--------------------------------------------------------------------------------

### Step 2: (replace KAFKA_OPTS configMap)

KAFKA_OPTS environment variable is update to cover log4j and Jolokia:

```
kubectl replace -f 02-kafka-brokers-opts-configmap.yml
```

--------------------------------------------------------------------------------

### Step 3: (Jolokia configMap)

Ensure to have deployed the jolokia.jar, the easiest is using a configMap:

```
kubectl create -f ../../Jolokia/01-jolokia-jar-configmap.yml
```

--------------------------------------------------------------------------------

### Step 4: (patch)

- Update the file 03-patch-kafka-connect.yml and 04-patch-kafka-connect.yml to match the name of your deployment:

```
kubectl -n kafka get deployments.apps
```

*Note: in sample, default used is confluent-oss-cp-kafka-connect*

```
kubectl --namespace kafka patch deployment confluent-oss-cp-kafka-connect --patch "$(cat 04-patch-kafka-connect.yml )"
```

--------------------------------------------------------------------------------

**To troubleshoot, useful kubectl commands:**

```
kubectl -n kafka describe deployments.apps confluent-oss-cp-kafka-connect
kubectl -n kafka get po
kubectl -n kafka describe po confluent-oss-cp-kafka-connect-64d4544f58-7tb9w
kubectl -n kafka logs confluent-oss-cp-kafka-connect-64d4544f58-7tb9w -c telegraf
kubectl -n kafka logs confluent-oss-cp-kafka-connect-64d4544f58-7tb9w -c cp-kafka-connect-server
```

--------------
[Go back](../)