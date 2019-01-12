# Telegraf sidecar container to monitor the LinkedIn Kafka monitor in a Kubernetes deployment

**The Kafka monitor by LinkedIn is an extremely powerful open source end to end monitoring solution for Kafka, please consult:**

https://github.com/linkedin/kafka-monitor

As a builtin configuration, the kafka-monitor implements a jolokia agent, so collecting the metrics with Telegraf only requires an input configuration within a Telegraf container.

--------------------------------------------------------------------------------

### Step 1: (Splunk url and token environment variables)

- Ensure you have created a configMap to reference the environment name, Splunk HEC url and token values that will be used by all your pods:

*../yaml_git_ignored/global-config.yml:*

```
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: kafka
  name: global-config
data:
  env: my-environment
  label: my-environment-label
  splunk_hec_url: my-splunk-hec.domain.com:8088
  splunk_hec_token: 205d43f1-2a31-4e60-a8b3-327eda49944a
```

*Create:*

```
kubectl create -f ../yaml_git_ignored/global-config.yml
```

The "splunk_hec_url" and "splunk_hec_token" are automatically substituted by the according values.

--------------------------------------------------------------------------------

### Step 2: Create your docker image, update and create the confiMap and start your Deployment

It is very straightforward to run the kafka-monitor in a docker container, first you need to create your own image:

https://github.com/linkedin/kafka-monitor/tree/master/docker

You can use the following public image on docker hub:

* guilhemmarchand/kafka-monitor:latest

**Verify and modify the yaml file 01-linkedin-kafka-monitor-configmap.yml to match your environment and needs, then create:**

```
kubectl create -f 01-linkedin-kafka-monitor-configmap.yml
```

- Create the Kubernetes Deployment:

```
kubectl create -f 02-linkedin-kafka-monitor-deployment.yml
```

--------------------------------------------------------------------------------

### Step 3: Create the configMap for Telegraf

```
kubectl create -f 03-telegraf-kafka-monitor-configmap.yml
```

--------------------------------------------------------------------------------

### Step 4: (patch for telegraf)

Finally patch your deployment to start monitoring:

```
kubectl --namespace kafka patch deployment kafka-monitor --patch "$(cat 04-patch-kafka-monitor.yml )"
```

**To troubleshoot, useful kubectl commands:**

```
kubectl -n kafka describe deployments.apps kafka-monitor
kubectl -n kafka get po
kubectl -n kafka describe po kafka-monitor-59994f9dcd-6t4vv
kubectl -n kafka logs kafka-monitor-59994f9dcd-6t4vv -c telegraf
kubectl -n kafka logs kafka-monitor-59994f9dcd-6t4vv -c kafka-monitor
```

--------------
[Go back](../)