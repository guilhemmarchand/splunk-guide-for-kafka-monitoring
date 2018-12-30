# Telegraf sidecar container to monitor Zookeeper in Kubernetes statefulSet

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
kubectl create -f 01-telegraf-config-zookeeper.yml
```

--------------------------------------------------------------------------------

### Step 2: (patch for telegraf)

The patch will update your Zookeeper statefulSet deployment and create the Splunk Universal Forwarder sidecar container.

- Update the file 02-patch-zookeeper-statefulset.yml to match the name of your statefulSet deployment

*This part must be changed to match the name of your statefulSet deployment:*

```
metadata:
  name: confluent-oss-cp-zookeeper
```

*This part must be changed to match the name of the container:*

```
      containers:
      - name: cp-zookeeper-server
```

- Run the patch command and ensure you specify the name of your statefulSet deployment:

```
kubectl --namespace kafka patch statefulset confluent-oss-cp-zookeeper --patch "$(cat 02-patch-zookeeper-statefulset.yml )"
```

--------------------------------------------------------------------------------

**To troubleshoot, useful kubectl commands:**

```
kubectl -n kafka describe statefulSet.apps confluent-oss-cp-zookeeper
kubectl -n kafka get po
kubectl -n kafka describe po confluent-oss-cp-zoopeeker-0
kubectl -n kafka logs confluent-oss-cp-zoopeeker-0 -c telegraf
kubectl -n kafka logs confluent-oss-cp-zookeeper-0 -c cp-zookeeper-server
kubectl -n kafka exec -it confluent-oss-cp-zookeeper-0 /bin/bash -c telegraf
kubectl -n kafka exec -it confluent-oss-cp-zookeeper-0 /bin/bash -c cp-zookeeper-server
```

--------------
[Go back](../)