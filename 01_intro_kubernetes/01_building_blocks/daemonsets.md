# DaemonSet

DaemonSets are operators designed to manage node agents. They resemble ReplicaSet and Deployment operators when managing multiple Pod replicas and application updates, but the DaemonSets present a distinct feature that enforces a single Pod replica to be placed per Node, on all the Nodes or on a select subset of Nodes. In contrast, the ReplicaSet and Deployment operators by default have no control over the scheduling and placement of multiple Pod replicas on the same Node.

``` yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-agent
  namespace: default
  labels:
    k8s-app: fluentd-agent
spec:
  selector:
    matchLabels:
      k8s-app: fluentd-agent
  template:
    metadata:
      labels:
        k8s-app: fluentd-agent
    spec:
      containers:
      - name: fluentd
        image: quay.io/fluentd_elasticsearch/fluentd:v4.5.2

```

    $ kubectl create -f fluentd-ds.yaml

Before advancing to more complex topics, become familiar with DaemonSet operations with additional commands such as:

    $ kubectl apply -f fluentd-ds.yaml --record
    $ kubectl get daemonsets
    $ kubectl get ds -o wide
    $ kubectl get ds fluentd-agent -o yaml
    $ kubectl get ds fluentd-agent -o json
    $ kubectl describe ds fluentd-agent
    $ kubectl rollout status ds fluentd-agent
    $ kubectl rollout history ds fluentd-agent
    $ kubectl rollout history ds fluentd-agent --revision=1
    $ kubectl set image ds fluentd-agent fluentd=quay.io/fluentd_elasticsearch/fluentd:v4.6.2 --record
    $ kubectl rollout history ds fluentd-agent --revision=2
    $ kubectl rollout undo ds fluentd-agent --to-revision=1
    $ kubectl get all -l k8s-app=fluentd-agent -o wide
    $ kubectl delete ds fluentd-agent
    $ kubectl get ds,po -l k8s-app=fluentd-agent