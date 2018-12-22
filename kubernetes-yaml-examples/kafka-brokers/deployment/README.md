# Telegraf deployment container to monitor Zookeeper in Kubernetes

**See:**

https://splunk-guide-for-kafka-monitoring.readthedocs.io/en/latest/chapter2_metrics.html#option-2-telegraf-as-a-deployment

**Step 1:**

- Expose the Zookeeper endpoints: (adapt depending on your pods names, namespace)

```
kubectl expose -n kafka pod kafka-0 --name kafka-0-jolokia-svc --port 8778
kubectl expose -n kafka pod kafka-1 --name kafka-1-jolokia-svc --port 8778
kubectl expose -n kafka pod kafka-2 --name kafka-2-jolokia-svc --port 8778
```

**Step 2:**

- Modify 01-telegraf-deployment-for-kafka-brokers.yml to match your Splunk HEC endpoint and namespace

- Create the deployment:

```
kubectl create -f 01-telegraf-deployment-for-kafka-brokers.yml
```

[../](Go back)