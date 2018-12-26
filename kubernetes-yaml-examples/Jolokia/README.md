# Configuration examples to deploy Jolokia for Kubernetes

--------------------------------------------------------------------------------

## Option 1: Jolokia jar in configMap

See:

https://splunk-guide-for-kafka-monitoring.readthedocs.io/en/latest/chapter2_metrics.html#option-1-jolokia-jar-in-configmap

*Download:*

```
curl http://search.maven.org/remotecontent?filepath=org/jolokia/jolokia-jvm/1.6.0/jolokia-jvm-1.6.0-agent.jar -o jolokia.jar
```

*Create the config map:*

```
kubectl create configmap jolokia-jar --from-file=jolokia.jar
```

*Export as a yaml:*

```
kubectl get configmaps jolokia-jar -o yaml --export > 01-jolokia-jar-configmap.yml
```

--------------------------------------------------------------------------------

## Option 2: NFS persistent volume configuration example

See

https://splunk-guide-for-kafka-monitoring.readthedocs.io/en/latest/chapter2_metrics.html#option-2-nfs-persistent-volume-configuration-example

--------------------------------------------------------------------------------

## Option 3: Local persistent volume configuration example

See

https://splunk-guide-for-kafka-monitoring.readthedocs.io/en/latest/chapter2_metrics.html#option-3-local-persistent-volume-configuration-example

--------------
[Go back](../)