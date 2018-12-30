# Telegraf sidecar container to monitor Confluent ksql-server in Kubernetes deployment

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
kubectl create -f 01-telegraf-config-confluent-ksql-server.yml
```

--------------------------------------------------------------------------------

### Step 2: (KSQL_OPTS configMap)

KSQL_OPTS environment variable is created Jolokia:

```
kubectl create -f 02-confluent-ksql-server-opts-configmap.yml
```

--------------------------------------------------------------------------------

### Step 3: (Jolokia configMap)

Ensure to have deployed the jolokia.jar, the easiest is using a configMap:

```
kubectl create -f ../../Jolokia/01-jolokia-jar-configmap.yml
```

--------------------------------------------------------------------------------

### Step 4: (patch)

The patch will update your Confluent ksql-server Deployment and create the Telegraf sidecar container.

- Update the file 04-patch-confluent-ksql-server.yml to match the name of your Deployment

*This part must be changed to match the name of your Deployment:*

```
metadata:
  name: confluent-oss-cp-ksql-server
```

*This part must be changed to match the name of the container:*

```
      containers:
      - name: cp-ksql-server
```

- Run the patch command and ensure you specify the name of your Deployment:

```
kubectl --namespace kafka patch deployment confluent-oss-cp-ksql-server --patch "$(cat 04-patch-confluent-ksql-server.yml )"
```

--------------------------------------------------------------------------------

**To troubleshoot, useful kubectl commands:**

```
kubectl -n kafka describe deployments.apps confluent-oss-cp-ksql-server
kubectl -n kafka get po
kubectl -n kafka describe po confluent-oss-cp-ksql-server-7cbbbddc7b-shz6l
kubectl -n kafka logs confluent-oss-cp-ksql-server-7cbbbddc7b-shz6l -c telegraf
kubectl -n kafka logs confluent-oss-cp-ksql-server-7cbbbddc7b-shz6l -c cp-ksql-server
```

--------------
[Go back](../)