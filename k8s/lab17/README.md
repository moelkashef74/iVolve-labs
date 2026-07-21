# Lab 17: Pod Resource Management with CPU and Memory Requests and Limits

## Lab Requirements

- Update the existing `nodejs-app` Deployment to include CPU and Memory resource requests and limits.
- Configure the following **Resource Requests**:
  - CPU: `1`
  - Memory: `1Gi`
- Configure the following **Resource Limits**:
  - CPU: `2`
  - Memory: `2Gi`
- Verify the configured resource requests and limits using `kubectl describe pod`.
- Monitor the pod's real-time resource usage using `kubectl top pod`.

---

# Requirements from Previous Labs

Before starting this lab, ensure the following resources already exist:

- A running Kubernetes 2-node cluster.

- A namespace named `ivolve`.

```bash
kubectl create namespace ivolve
```

- A ResourceQuota named `app-quota` in the `ivolve` namespace.

```bash
kubectl apply -f k8s/lab11/quota.yaml
```

- A worker node tainted with:

```bash
kubectl taint nodes node01 node=worker:NoSchedule
```

- A ConfigMap containing the application configuration.

```bash
kubectl apply -f k8s/lab12/app-config.yaml
```

- A Secret containing the application's sensitive information.

```bash
kubectl apply -f k8s/lab12/app-secrets.yaml
```

- A statically provisioned PersistentVolume and PersistentVolumeClaim.

```bash
kubectl apply -f k8s/lab13/pv-static.yaml
kubectl apply -f k8s/lab13/pvc-static.yaml
```

- Your custom Node.js Docker image pushed to Docker Hub.

- The Node.js application Service.

```bash
kubectl apply -f k8s/lab15/nodejs-svc.yaml
```

- MySQL StatefulSet and Headless Service.

```bash
kubectl apply -f k8s/lab14/sql-statefulset.yaml
kubectl apply -f k8s/lab14/headless-svc.yaml
```

- The updated Node.js Deployment with the Init Container from Lab 16.

---

# Step 1: Update the Deployment

## Edit **nodejs-deployment.yaml** and add the following `resources` section under the `nodejs-app` container.

```yaml
resources:
  requests:
    cpu: "1"
    memory: "1Gi"
  limits:
    cpu: "2"
    memory: "2Gi"
```

The container definition should look similar to the following:

```yaml
containers:
- name: nodejs-app
  image: mosayed711/nodejs-app:v2

  envFrom:
  - configMapRef:
      name: app-env

  env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secrets
        key: DB_PASSWORD

  resources:
    requests:
      cpu: "1"
      memory: "1Gi"
    limits:
      cpu: "2"
      memory: "2Gi"

  volumeMounts:
  - name: app-logs
    mountPath: /var/log/app
```

 ## Apply the updated Deployment.

```bash
kubectl apply -f nodejs-deployment.yaml
```

---

## Verify the Deployment

### Verify that the pods are running successfully.

```bash
kubectl get pods -n ivolve
```

Output:

```text
NAME                          READY   STATUS    RESTARTS   AGE
mysql-0                       1/1     Running   0          7m52s
nodejs-app-546fccf968-kvhsb   1/1     Running   0          7m25s
```

---

# Step 2 : Verify Resource Requests and Limits

Describe one of the running pods.

```bash
kubectl describe pod nodejs-app-546fccf968-kvhsb -n ivolve
```

Under the **Containers** section, verify the configured resource requests and limits.

output:

```text
Limits:
  cpu:     2
  memory:  2Gi

Requests:
  cpu:     1
  memory:  1Gi
```

---

# Step 3: Monitor Resource Usage

Ensure the Metrics Server is installed.

Verify:

```bash
kubectl top nodes
```
```text
NAME           CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)   
controlplane   149m         14%      1332Mi          62%         
node01         131m         13%      930Mi           51%
```


```bash
kubectl top pod -n ivolve
```

Example output:

```text
NAME                          CPU(cores)   MEMORY(bytes)   
mysql-0                       1m           160Mi           
nodejs-app-546fccf968-s2zf4   1m           20Mi   
```

---
