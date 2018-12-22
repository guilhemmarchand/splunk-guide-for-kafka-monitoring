# Configuration examples to deploy Jolokia for Kubernetes

## Option 1: Jolokia jar in configMap

**See: https://splunk-guide-for-kafka-monitoring.readthedocs.io/en/latest/chapter2_metrics.html#option-1-jolokia-jar-in-configmap**

```
curl http://search.maven.org/remotecontent?filepath=org/jolokia/jolokia-jvm/1.6.0/jolokia-jvm-1.6.0-agent.jar -o jolokia.jar
```

```
kubectl create configmap jolokia-jar --from-file=jolokia.jar
```

```
kubectl get configmaps jolokia-jar -o yaml --export > jolokia-jar-configMap.yml
```
