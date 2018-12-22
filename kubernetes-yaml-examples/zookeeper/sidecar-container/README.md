# Telegraf sidecar container to monitor Zookeeper in Kubernetes statefulSet

**See:**

https://splunk-guide-for-kafka-monitoring.readthedocs.io/en/latest/chapter2_metrics.html#option-1-telegraf-as-a-sidecar-container

**Step 1:**

- Modify 01-telegraf-config-zookeeper.yml to match your Splunk HEC endpoint URL

- Create the Kubernetes configMap:

```
kubectl create -f 01-telegraf-config-zookeeper.yml
```

**Step 2:**

- Modify 02-patch-zookeeper-statefulset.yml to match the name of your statefulSet (default named zookeeper)

- Patch your statefulSet (modify the name of your statefulSet and namespace in the kubectl command line if different):

```
kubectl --namespace kafka patch statefulset zookeeper --patch "$(cat 02-patch-zookeeper-statefulset.yml )"
```

--------------
[Go back](../)