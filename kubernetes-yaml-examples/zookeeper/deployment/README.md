# Telegraf deployment container to monitor Zookeeper in Kubernetes

**See:**

https://splunk-guide-for-kafka-monitoring.readthedocs.io/en/latest/chapter2_metrics.html#option-2-telegraf-as-a-deployment

**Step 1:**

- Expose the Zookeeper endpoints: (adapt depending on your pods names, namespace)

```
kubectl expose -n kafka pod zookeeper-0 --name zookeeper-0-svc --port 2181
kubectl expose -n kafka pod zookeeper-1 --name zookeeper-1-svc --port 2181
kubectl expose -n kafka pod zookeeper-2 --name zookeeper-2-svc --port 2181
```

**Step 2:**

- Modify 01-telegraf-deployment-for-zookeeper.yml to match your Splunk HEC endpoint and namespace

- Create the deployment:

```
kubectl create -f 01-telegraf-deployment-for-zookeeper.yml
```

--------------
[Go back](../)