## Annotations

 Annotations allow us to attach arbitrary non-identifying metadata to any objects, in a key-value format:

```yaml
"annotations": {
  "key1": "value1",
  "key2": "value2"
}
```

We can easily annotate an existing object, a Pod for example, as such:

```
$ kubectl annotate pod mypod key1=value1 key2=value2
```

Unlike Labels, annotations are not used to identify and select objects. Annotations can be used to:

- Store build/release IDs, PR numbers, git branch, etc.
- Phone/pager numbers of people responsible, or directory entries specifying where such information can be found.
- Pointers to logging, monitoring, analytics, audit repositories, debugging tools, etc.
- Ingress controller information.
- Deployment state and revision information.

## Quota and Limits Management

When there are many users sharing a given Kubernetes cluster, there is always a concern for fair usage. A user should not take undue advantage. To address this concern, administrators can use the ResourceQuota API resource, which provides constraints that limit aggregate resource consumption per Namespace.
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
  namespace: myspace
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    requests.nvidia.com/gpu: 4:
```    
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-counts
  namespace: myspace
spec:
  hard:
    configmaps: "10"
    persistentvolumeclaims: "4"
    pods: "4"
    secrets: "10"
    services: "10"
    services.loadbalancers: "2"
```    

### LimitRange

Examine the LimitRange manifest below. It exemplifies how CPU constraints are enforced for individual Containers of Pods running in the myspace namespace:
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-constraint-per-container
  namespace: myspace
spec:
  limits:
  - default:            # default limits
      cpu: 500m
    defaultRequest:     # default requests
      cpu: 500m
    max:                # max defines the highest value of the range
      cpu: "1"
    min:                # min defines the lowest value of the range
      cpu: 100m
    type: Container
```

## Autoscaling

Autoscaling can be implemented in a Kubernetes cluster via controllers which periodically adjust the number of running objects based on single, multiple, or custom metrics. There are various types of autoscalers available in Kubernetes which can be implemented individually or combined for a more robust autoscaling solution:

- **Horizontal Pod Autoscaler (HPA)**  
    HPA is an algorithm-based controller API resource which automatically adjusts the number of replicas in a ReplicaSet, Deployment, or Replication Controller based on CPU utilization. An easy method to deploy an HPA object is through the following imperative command. This HPA dynamically triggers the scaling of a myapp Deployment when CPU reaches 80% utilization, between 2 and 10 replicas: kubectl autoscale deploy myapp --min=2 --max=10 --cpu-percent=80

- **Vertical Pod Autoscaler (VPA)**
    VPA automatically sets Container resource requirements (CPU and memory) in a Pod and dynamically adjusts them at runtime, based on historical utilization data, current resource availability and real-time events. It is installed as a Custom Resource.

- **Cluster Autoscaler**
    Cluster Autoscaler automatically re-sizes the Kubernetes cluster when there are insufficient resources available for new Pods expecting to be scheduled or when there are underutilized nodes in the cluster.

## Jobs and CronJobs

A Job creates one or more Pods to perform a given task. The Job object takes the responsibility of Pod failures.

Starting with the Kubernetes 1.4 release, we can also perform Jobs at scheduled times/dates with CronJobs, where a new Job object is created about once per each execution cycle. 

## StatefulSets
The StatefulSet controller is used for stateful applications which require a unique identity, such as name, network identifications, or strict ordering, such as a MySQL cluster.

The StatefulSet controller provides identity and guaranteed ordering of deployment and scaling to Pods. However, the StatefulSet controller has very strict Service and Storage Volume dependencies that make it challenging to configure and deploy. It also supports scaling, rolling updates, and rollbacks.

## Custom Resources
In Kubernetes, a resource is an API endpoint which stores a collection of API objects. For example, a Pod resource contains all the Pod objects.

Although in most cases existing Kubernetes resources are sufficient to fulfill our requirements, we can also create new resources using custom resources. With custom resources, we don't have to modify the Kubernetes source.

## Security Contexts 

At times we need to define specific privileges and access control settings for Pods and Containers. Security Contexts allow us to set Discretionary Access Control for object access permissions, privileged running, capabilities, security labels, etc. However, their effect is limited to the individual Pods and Containers where such context configuration settings are incorporated in the spec section.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
  - name: vol
    emptyDir: {}
  containers:
  - name: busy
    image: busybox:1.28
    command: [ "sh", "-c", "sleep infinity" ]
    volumeMounts:
    - name: vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: false
```


## Pod Security Admission

In order to apply security settings to multiple Pods and Containers cluster-wide, we can use the Pod Security Admission, a built-in admission controller for Pod Security that is enabled by default in the API Server. It can enforce the three Pod Security Standards at namespace level, by automating the security context restriction to Pods when they are deployed. Each Pod Security Standard profile, privileged, baseline, and restricted, defines a level of security that ranges from entirely unrestricted (for privileged workload), to enforcing Pod hardening best practices (for security critical applications and less trusted users). The security levels are defined by sets of Pod security context controls that are pre-determined for each standard.


## Network Policies

Kubernetes was designed to allow all Pods to communicate freely, without restrictions, with all other Pods in cluster Namespaces. In time it became clear that it was not an ideal design, and mechanisms needed to be put in place in order to restrict communication between certain Pods and applications in the cluster Namespace. Network Policies are sets of rules which define how Pods are allowed to talk to other Pods and resources inside and outside the cluster. Pods not covered by any Network Policy will continue to receive unrestricted traffic from any endpoint.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: demo-netpol
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

## Monitoring, Logging, and Troubleshooting

In Kubernetes, we have to collect resource usage data by Pods, Services, nodes, etc., to understand the overall resource consumption and to make decisions for scaling a given application. Two popular Kubernetes monitoring solutions are the Kubernetes Metrics Server and Prometheus. 

- **Metrics Server**  
    Metrics Server is a cluster-wide aggregator of resource usage data - a relatively new feature in Kubernetes, available as a plugin. With the metrics-server installed, we can easily extract resource utilization data from the cluster with the help of the kubectl top nodes and kubectl top pods commands. Both commands are namespace specific and support sorting with the help of the --sort-by=cpu or memory options.

- **Prometheus** 
    Prometheus, now a graduated project of CNCF (Cloud Native Computing Foundation), can also be used to scrape the resource usage from different Kubernetes components and objects. Using its client libraries, we can also instrument the code of our application.

Another important aspect for troubleshooting and debugging is logging, in which we collect the logs from different components of a given system. In Kubernetes, we can collect logs from different cluster components, objects, nodes, etc. Unfortunately, however, Kubernetes does not provide cluster-wide logging by default, therefore third party tools are required to centralize and aggregate cluster logs. A popular method to collect logs is using Elasticsearch together with Fluentd with custom configuration as an agent on the nodes. Fluentd is an open source data collector, which is also a graduated project of CNCF.

```
$ kubectl logs pod-name

$ kubectl logs pod-name container-name

$ kubectl logs pod-name container-name -p

$ kubectl exec pod-name -- ls -la /

$ kubectl exec pod-name -c container-name -- env

$ kubectl exec pod-name -c container-name -it -- /bin/sh

$ kubectl get events

$ kubectl events

$ kubectl describe pod pod-name
```

## Helm

To deploy a complex application, we use a large number of Kubernetes manifests to define API resources such as Deployments, Services, PersistentVolumes, PersistentVolumeClaims, Ingress, or ServiceAccounts. It can become counter productive to deploy them one by one. We can bundle all those manifests after templatizing them into a well-defined format, along with other metadata. Such a bundle is referred to as Chart. These Charts can then be served via repositories, such as the ones for rpm and deb packages, or the container image registries.

Helm is a package manager (analogous to yum and apt for Linux) for Kubernetes, which can install/update/delete those Charts in the Kubernetes cluster.

## Service Mesh

Service Mesh is a third party solution alternative to the Kubernetes native application connectivity and exposure achieved with Services paired with Ingress Controllers. Service Mesh tools are gaining popularity especially with larger organizations managing larger, more dynamic Kubernetes clusters. These third party solutions introduce features such as service discovery, mutual TLS (mTLS) certificates for encryption, multi-cloud routing, and traffic telemetry.

A Service Mesh is an implementation that relies on a proxy component part of the Data Plane, which is then managed through a Control Plane. The Control Plane runs agents responsible for the service discovery, telemetry, load balancing, network policy, and optionally ingress and/or egress gateway.

- Consul by HashiCorp
- Istio is one of the most popular service mesh solutions, backed by Google, IBM and Lyft
- Kuma by Kong
- Linkerd a CNCF project
- Cilium an eBPF-based service mesh
- Traefik Mesh by Containous, the developers of Traefik proxy ingress controller
- Tanzu Service Mesh by VMware

## Application Deployment Strategies

A method presented earlier for new application release rollouts was the Rolling Update mechanism supported by the Deployment operator. The Rolling Update mechanism, and its reverse - the Rollback, are practical methods to manage application updates by allowing one single controller, the Deployment, to handle all the work it involves. However, while transitioning between the old and the new versions of the application replicas, the Service exposing the Deployment eventually forwards traffic to all replicas, old and new, without any possibility for the default Service to isolate a subset of the Deployment's replicas. Because of the traffic routing challenges these update mechanisms introduce, many users may steer away from the one Deployment and one Service model, and embrace more complex deployment mechanism alternatives.

The Canary strategy runs two application releases simultaneously managed by two independent Deployment controllers, both exposed by the same Service. The users can manage the amount of traffic each Deployment is exposed to by separately scaling up or down the two Deployment controllers, thus increasing or decreasing the number of their replicas receiving traffic.

The Blue/Green strategy runs the same application release or two releases of the application on two isolated environments, but only one of the two environments is actively receiving traffic, while the second environment is idle, or may undergo rigorous testing prior to shifting traffic to it. This strategy would also require two independent Deployment controllers, each exposed by their dedicated Services, however, a traffic shifting mechanism is also required. Typically, the traffic shifting can be implemented with the use of an Ingress or another Service.

