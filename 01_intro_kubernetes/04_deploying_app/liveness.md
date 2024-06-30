# Liveness, Readiness, and Startup Probes

### Liveness

Liveness Probe checks on an application's health, and if the health check fails, kubelet restarts the affected container automatically.

**Liveness Probes can be set by defining:**
- Liveness command
- Liveness HTTP request
- TCP Liveness probe
- gRPC Liveness probe

### Liveness Command

In the following example, the liveness command is checking the existence of a file /tmp/healthy:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 15
      failureThreshold: 1
      periodSeconds: 5
```

### Liveness HTTP Request

In the following example, the kubelet sends the HTTP GET request to the /healthz endpoint of the application, on port 8080. If that returns a failure, then the kubelet will restart the affected container; otherwise, it would consider the application to be alive:

```yaml
    livenessProbe:
    httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: X-Custom-Header
        value: Awesome
    initialDelaySeconds: 15
    periodSeconds: 5
```

### TCP Liveness Probe

With TCP Liveness Probe, the kubelet attempts to open the TCP Socket to the container running the application. If it succeeds, the application is considered healthy, otherwise the kubelet would mark it as unhealthy and restart the affected container.

```yaml
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 5
```

### gRPC Liveness Probe

The gRPC Liveness Probe can be used for applications implementing the gRPC health checking protocol. It requires for a port to be defined, and optionally a service field may help adapt the probe for liveness or readiness by allowing the use of the same port.

```yaml
    livenessProbe:
      grpc:
        port: 2379
      initialDelaySeconds: 10
```

### Readiness Probes
Sometimes, while initializing, applications have to meet certain conditions before they become ready to serve traffic. These conditions include ensuring that the dependent service is ready, or acknowledging that a large dataset needs to be loaded, etc. In such cases, we use Readiness Probes and wait for a certain condition to occur. Only then, the application can serve traffic.

A Pod with containers that do not report ready status will not receive traffic from Kubernetes Services.

```yaml
    readinessProbe:
          exec:
            command:
            - cat
            - /tmp/healthy
          initialDelaySeconds: 5 
          periodSeconds: 5
```

### Startup Probes
The newest member of the Probes family is the Startup Probe. This probe was designed for legacy applications that may need more time to fully initialize and its purpose is to delay the Liveness and Readiness probes, a delay long enough to allow for the application to fully initialize.

