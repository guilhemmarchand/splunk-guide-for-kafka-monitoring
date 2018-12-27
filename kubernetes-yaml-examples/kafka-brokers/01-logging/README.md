# Splunk Universal Forwarder sidecar container to monitor Kafka brokers in Kubernetes statefulSet

**See:**

--------------------------------------------------------------------------------

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

### Step 2: (KAFKA_OPTS configMap)

*Create:*

```
kubectl create -f 02-kafka-brokers-opts-configmap.yml
```

--------------------------------------------------------------------------------

### Step 3: (log4j configMap)

*Create:*

```
kubectl create -f 03-kafka-brokers-log4j-configmap.yml

--------------------------------------------------------------------------------

### Step 4: (patch)

- Update the file 04-patch-shared-volume-and-splunk-uf.yml to match the name of your statefulSet deployment:

kubectl --namespace kafka patch statefulset confluent-oss-cp-kafka --patch "$(cat 04-patch-shared-volume-and-splunk-uf.yml )"
