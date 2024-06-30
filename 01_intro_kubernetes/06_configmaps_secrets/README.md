# ConfigMaps and Secrets

## ConfigMaps

Configmaps allow us to decouple the configuration details from the container image. Using ConfigMaps, we pass configuration data as key-value pairs, which are consumed by Pods or any other system components and controllers, in the form of environment variables, sets of commands and arguments, or volumes. We can create ConfigMaps from literal values, from configuration files, from one or more files or directories.

### Create a ConfigMap from Literal Values

```
$ kubectl create configmap my-config \
  --from-literal=key1=value1 \
  --from-literal=key2=value2

$ kubectl get configmaps my-config -o yaml
```
```yaml
apiVersion: v1
data:
  key1: value1
  key2: value2
kind: ConfigMap
metadata:
  creationTimestamp: 2024-03-02T07:21:55Z
  name: my-config
  namespace: default
  resourceVersion: "241345"
  selfLink: /api/v1/namespaces/default/configmaps/my-config
  uid: d35f0a3d-45d1-11e7-9e62-080027a46057
  ```
  
  We can then create the ConfigMap from a file

```
$ kubectl create configmap permission-config \
  --from-file=<path/to/>permission-reset.properties
```

### Create a ConfigMap from a Definition Manifest

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: customer1
data:
  TEXT1: Customer1_Company
  TEXT2: Welcomes You
  COMPANY: Customer1 Company Technology Pct. Ltd.
```
```
$ kubectl create -f customer1-configmap.yaml
```

### Create a ConfigMap from a File

Create a file permission-reset.properties
```
permission=read-only
allowed="true"
resetCount=3
```
We can then create the ConfigMap with the following command:
```
$ kubectl create configmap permission-config
```

### Use ConfigMaps Inside Pods: As Environment Variables

In the following example all the myapp-full-container Container's environment variables receive the values of the full-config-map ConfigMap keys:

```yaml
  containers:
  - name: myapp-full-container
    image: myapp
    envFrom:
    - configMapRef:
      name: full-config-map
```

In the following example the myapp-specific-container Container's environment variables receive their values from specific key-value pairs from two separate ConfigMaps, config-map-1 and config-map-2 respectively:

```yaml
  containers:
  - name: myapp-specific-container
    image: myapp
    env:
    - name: SPECIFIC_ENV_VAR1
      valueFrom:
        configMapKeyRef:
          name: config-map-1
          key: SPECIFIC_DATA
    - name: SPECIFIC_ENV_VAR2
      valueFrom:
        configMapKeyRef:
          name: config-map-2
          key: SPECIFIC_INFO
```

### Use ConfigMaps Inside Pods: As Volumes

We can mount a vol-config-map ConfigMap as a Volume inside a Pod. The configMap Volume plugin converts the ConfigMap object into a mountable resource. For each key in the ConfigMap, a file gets created in the mount path (where the file is named with the key name) and the respective key's value becomes the content of the file:

```yaml
  containers:
  - name: myapp-vol-container
    image: myapp
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: vol-config-map
```

## Secrets

### Create a Secret from Literal Values

```
$ kubectl create secret generic my-password \
--from-literal=password=mysqlpassword`
```

### Create a Secret from a Definition Manifest


With **data maps**, each value of a sensitive information field must be encoded using base64.

```
$ echo mysqlpassword | base64

bXlzcWxwYXNzd29yZAo=
```
and then use it in the definition manifest:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-password
type: Opaque
data:
  password: bXlzcWxwYXNzd29yZAo=
```

With **stringData** maps, there is no need to encode the value of each sensitive information field.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-password
type: Opaque
stringData:
  password: mysqlpassword

```

### Create a Secret from a File

To create a Secret from a File, we can use the kubectl create secret command. 

First, we encode the sensitive data and then we write the encoded data to a text file:
```
$ echo mysqlpassword | base64

bXlzcWxwYXNzd29yZAo=

$ echo -n 'bXlzcWxwYXNzd29yZAo=' > password.txt

$ kubectl create secret generic my-file-password \
  --from-file=password.txt
```
### Use Secrets Inside Pods: As Environment Variables

Secrets are consumed by Containers in Pods as mounted data volumes, or as environment variables, and are referenced in their entirety (using the envFrom heading) or specific key-values (using the env heading).

```yaml
spec:
  containers:
  - image: wordpress:4.7.3-apache
    name: wordpress
    env:
    - name: WORDPRESS_DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-password
          key: password
```

### Use Secrets Inside Pods: As Volumes

We can also mount a Secret as a Volume inside a Pod. The secret Volume plugin converts the Secret object into a mountable resource. The following example creates a file for each my-password Secret key (where the files are named after the names of the keys), the files containing the values of the respective Secret keys:

```yaml
spec:
  containers:
  - image: wordpress:4.7.3-apache
    name: wordpress
    volumeMounts:
    - name: secret-volume
      mountPath: "/etc/secret-data"
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: my-password
```

For m