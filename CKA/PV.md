# Persistence Volumne & Persistence Volume Claim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myClaim
spec:
  accessModes:
    - ReadWriteOne
  resources:
    requests:
      storage: 500Mi
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    stroage: 1Gi
  awsElasticBlockStorage:
    volumeId: <volume-id>
    fsType: ext4
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/pv/log"
  persistentVolumeReclaimPolicy: Retain
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  namespace: default
spec:
  containers:
  - env:
    - name: LOG_HANDLERS
      value: file
    image: kodekloud/event-simulator
    name: event-simulator
    volumeMounts:
    - mountPath: /log
      name: claim-log-1
  volumes:
  - name: log-volume
    hostPath:
        path: /var/log/webapp
  - name: claim-log-1
    persistentVolumeClaim:
      claimName: claim-log-1
```
