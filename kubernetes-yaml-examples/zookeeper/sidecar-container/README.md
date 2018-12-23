# Telegraf sidecar container to monitor Zookeeper in Kubernetes statefulSet

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

*Create and apply your secrets:*

```
*../../yaml_git_ignored/splunk_secrets.yml*
```

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
kubectl create -f 01-telegraf-config-zookeeper.yml
```

**Step 2:**

- Update the file 02-patch-zookeeper-statefulset.yml to match the name of your statefulSet deployment:

```
kubectl -n kafka get kubectl -n kafka get statefulsets.apps
```

*Note: in sample, default used is confluent-oss-cp-zookeeper*

- Patch your statefulSet (modify the name of your statefulSet and namespace in the kubectl command line if different):

```
kubectl --namespace kafka patch statefulset confluent-oss-cp-zookeeper --patch "$(cat 02-patch-zookeeper-statefulset.yml )"
```

**To troubleshoot, useful kubectl commands:**

```
kubectl -n kafka describe statefulSet.apps confluent-oss-cp-zookeeper
kubectl -n kafka get po
kubectl -n kafka describe po confluent-oss-cp-zoopeeker-0
kubectl -n kafka logs confluent-oss-cp-zoopeeker-0 -c telegraf
kubectl -n kafka logs confluent-oss-cp-zookeeper-0 -c cp-zookeeper-server
kubectl -n kafka exec -it confluent-oss-cp-zookeeper-0 /bin/bash -c telegraf
```

--------------
[Go back](../)