# Lab 13: Persistent Storage Setup for Application Logging

## Lab Requirements

1. **Static Provisioning**
   - Manually create a PersistentVolume (PV).
   - Bind it to a PersistentVolumeClaim (PVC).

2. **Dynamic Provisioning**
   - Create a StorageClass.
   - Allow Kubernetes to automatically create PVs when a PVC is requested.

---

# Part 1: Static Provisioning (Manual PV Creation)
---

## Step 1: Create the hostPath directory

On the Kubernetes node:

```bash
mkdir -p /mnt/app-logs
```
---

## Step 2: Create the Persistent Volume (PV)

Create a file:

```bash
vim pv-static.yaml
```
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-logs-pv
spec:
  capacity:
    storage: 1Gi

  accessModes:
    - ReadWriteMany

  persistentVolumeReclaimPolicy: Retain

  storageClassName: ""

  hostPath:
    path: /mnt/app-logs
```

Apply:

```bash
kubectl apply -f pv-static.yaml
```

Verify:

```bash
kubectl get pv
```

Output:

```
NAME            CAPACITY   ACCESS MODES   STATUS
app-logs-pv     1Gi        RWX            Available
```

---

## Step 3: Create the Persistent Volume Claim (PVC)

Create:

```bash
vim pvc-static.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-logs-pvc
spec:
  accessModes:
    - ReadWriteMany

  storageClassName: ""

  resources:
    requests:
      storage: 1Gi
```

Apply:

```bash
kubectl apply -f pvc-static.yaml
```

---

## Step 4: Verify PV/PVC Binding

Check PVC:

```bash
kubectl get pv,pvc
```

output:

```
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM            STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
persistentvolume/app-logs-pv   1Gi        RWX            Retain           Bound    default/my-pvc                  <unset>                          81s

NAME                           STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/app-logs-pvc   Bound    pv0003   1Gi        RWX                           <unset>                 23s
```
---

# Part 2: Dynamic Provisioning

---

## Step 1: Create StorageClass

using Rancher Local Path Provisioner:

```bash
vim storageclass.yaml
```

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: app-logs-sc

provisioner:
  rancher.io/local-path

reclaimPolicy:
  Retain

volumeBindingMode:
  WaitForFirstConsumer
```

Apply:

```bash
kubectl apply -f storageclass.yaml
```
---

## Step 2: Create PVC Using StorageClass

Create:

```bash
vim pvc-dynamic.yaml
```

Add:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-logs-dynamic-pvc

spec:
  accessModes:
    - ReadWriteOnce

  storageClassName:
    app-logs-sc

  resources:
    requests:
      storage: 1Gi
```

Apply:

```bash
kubectl apply -f pvc-dynamic.yaml
```

---

## Step 3: Create Pod Using PVC

Create:

```bash
vim app.yaml
```

Add:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: logging-app

spec:
  containers:
  - name: app
    image: nginx

    volumeMounts:
    - name: logs
      mountPath: /var/log/app

  volumes:
  - name: logs
    persistentVolumeClaim:
      claimName: app-logs-dynamic-pvc
```

Apply:

```bash
kubectl apply -f app.yaml
```

---

## Step 4: Verify Dynamic PV Creation

Check PVC:

```bash
kubectl get pvc
```

output:

```
NAME                       STATUS
app-logs-dynamic-pvc       Bound
```

Check PV:

```bash
kubectl get pv,pvc
```

output
```bash
persistentvolume/pvc-005a1cf8-e251-4d3e-9c64-be1ee68d7f57   1Gi        RWO            Retain           Bound    default/app-logs-dynamic-pvc   app-logs-sc    <unset>                          3m16s

NAME                                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/app-logs-dynamic-pvc   Bound    pvc-005a1cf8-e251-4d3e-9c64-be1ee68d7f57   1Gi        RWO            app-logs-sc    <unset>                 3m22s

---
```
