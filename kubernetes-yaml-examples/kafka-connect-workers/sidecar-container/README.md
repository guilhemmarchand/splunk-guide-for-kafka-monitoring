# Telegraf sidecar container to monitor Kafka Connect in Kubernetes statefulSet

**See:**

https://splunk-guide-for-kafka-monitoring.readthedocs.io/en/latest/chapter2_metrics.html#option-1-telegraf-as-a-sidecar-container

**Step 1:**

- Ensure you have created secrets of your Splunk HEC url and token value, these secrets needs to be created ONCE ONLY for all containers, example:

*Generate the base64 values:*

```
echo "splunk_hec.mydomain.com" | base64
c3BsdW5rX2hlYy5teWRvbWFpbi5jb20K
echo -n '65735c4b-f277-4f69-87ca-ff2b738c69f9' | base64
NjU3MzVjNGItZjI3Ny00ZjY5LTg3Y2EtZmYyYjczOGM2OWY5
```

*Create and apply your secrets: ../../yaml_git_ignored/splunk_secrets.yml*

```
apiVersion: v1
kind: Secret
metadata:
  name: splunk-secrets
  namespace: kafka
type: Opaque
data:
  splunk_hec_url: c3BsdW5rX2hlYy5teWRvbWFpbi5jb20K
  splunk_hec_token: NjU3MzVjNGItZjI3Ny00ZjY5LTg3Y2EtZmYyYjczOGM2OWY5
```

*Create:*

```
kubectl create -f ../../yaml_git_ignored/splunk_secrets.yml
```

The "splunk_hec_url" and "splunk_hec_token" are automatically substituted by the according values.

- Create the Kubernetes configMap:

```
kubectl create -f 01-telegraf-config-kafka-connect.yml
```

**Step 2:**

Ensure to have deployed the jolokia.jar, the easiest is using a configMap:

```
kubectl create -f ../../Jolokia/01-jolokia-jar-configmap.yml
```

**Step 3:**

- Update the file 03-patch-kafka-connect-statefulset.yml and 04-patch-kafka-connect-statefulset.yml to match the name of your statefulSet deployment:

```
kubectl -n kafka get kubectl -n kafka get statefulsets.apps
```

*Note: in sample, default used is confluent-oss-cp-kafka-connect*

Modify manually your deployment to include the jolokia volume and the JVM starting agent, or even easier patch your existing statefulSet:

```
kubectl --namespace kafka patch deployment confluent-oss-cp-kafka-connect --patch "$(cat 03-patch-kafka-connect-statefulset.yml )"
```

**Step 4:**

Finally patch your statefulSet to start monitoring:

- Modify 04-patch-kafka-brokers-statefulset.yml to match the name of your statefulSet (default named zookeeper)

- Patch your statefulSet (modify the name of your statefulSet and namespace in the kubectl command line if different):

```
kubectl --namespace kafka patch deployment confluent-oss-cp-kafka-connect --patch "$(cat 04-patch-kafka-connect-statefulset.yml )"
```

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