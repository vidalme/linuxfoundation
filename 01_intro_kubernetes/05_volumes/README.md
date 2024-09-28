# KUBERNETES VOLUME MANAGEMENT

### Container Storage Interface (CSI)

Storage vendors and community members from different orchestrators started working together to standardize the Volume interface - a volume plugin built using a standardized Container Storage Interface (CSI) designed to work on different container orchestrators with a variety of storage providers.

### Volume Types


### PersistentVolumes

A Persistent Volume is a storage abstraction backed by several storage technologies, which could be local to the host where the Pod is deployed with its application container(s), network attached storage, cloud storage, or a distributed storage solution. A Persistent Volume is statically provisioned by the cluster administrator.

![alt text](image.png)

**nfs**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
    name: pv-name
spec:
    capactity:
        storage: 5Gi
    volumeMode: Filesystem
    accessModes:
        - ReadWriteOnce
    persistentVolumeReclaimPolicy: Recycle
    storageClassName: slow
    mountOptions:
        - hard
        - nfsvers=4.0
    nfs:
        path: /dir/path/on/nfs/server
        server: nfs-server-ip-address
```

**google cloud**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
    name: test-volume
    labels:
        failure-domain.beta.kubernetes.io/zone: us-certal1-a__us-central1-b
spec:
    capactity:
        storage: 400Gi
    accessModes:
        - ReadWriteOnce
    gcePersistentDisk:
        pdName: my-data-disk
        fsType: ext4
```

### PersistentVolumeClaim

**persistentvolumeclaim**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: pvc-name
spec:
    storageClassName: manual
    volumeMode: Filesystem
    accessMoeds:
        - ReadWriteOnce
    resources:
        requests:
            storage: 10Gi
```

**pod**
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: mypod
spec:
    containers:
        - name: myfrontend
          image: nginx
          volumeMounts:
          - mountPath: "/var/www/html"
            name: mypd
    volumes:
        - name: mypd
          persistentVolumeClaim:
            claimName: pvc-name
```