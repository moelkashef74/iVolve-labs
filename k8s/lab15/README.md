# Lab 15: Node.js Application Deployment with ClusterIP Service

## Lab Requirements

- Create a Deployment named `nodejs-app`.
- Configure the Deployment with **2 replicas** (only one pod will be running due to scheduling constraints).
- Use your custom Docker image from Docker Hub.
- Configure the application using environment variables from a ConfigMap and Secret.
- Configure the pod to tolerate the worker node taint.
- Mount the previously created static Persistent Volume.
- Create a ClusterIP Service named `nodejs-service`.
- Verify the application is accessible through the service.

---

# Requirements from Previous Labs

- A running Kubernetes 2-node cluster.
- A namespace named `ivolve`.
```bash
kubectl create namespace ivolve
```
- A resource quota named `app-quota` in the `ivolve` namespace.
```bash
kubectl apply -f k8s/lab1/quota.yaml
```

- A worker node tainted with:

```bash
kubectl taint nodes node01 node=worker:NoSchedule
```

- A ConfigMap for application configuration.

```bash
kubectl apply -f k8s/lab12/app-config.yaml
```

- A Secret for app sensitive information.

```bash
kubectl apply -f k8s/lab12/app-secret.yaml
```

- A statically provisioned PersistentVolume and PersistentVolumeClaim.

```bash
kubectl apply -f k8s/lab13/pv-static.yaml
kubectl apply -f k8s/lab13/pvc-static.yaml
```

- Your custom Node.js Docker image pushed to Docker Hub.

---
# Step 1: Create docker-cred secret to pull your private image

```bash
kubectl create secret docker-registry docker-cred \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=<your-dockerhub-username> \
  --docker-password=<your-dockerhub-password> \
  --docker-email=<your-email> --dry-run=client -o yaml > docker-cred.yaml
```
then edit the file to add the namespace
```bash
kubectl apply -f docker-cred.yaml
```

# Step 2: Create the Deployment

Create a file named **nodejs-deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-app
  namespace: ivolve
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nodejs-app
  template:
    metadata:
      labels:
        app: nodejs-app
    spec:
      dockerimagePullSecrets:
        name: docker-cred
      containers:
      - name: nodejs-app
        image: mosayed711/nodejs-app:v2
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-env
              key: DB_HOST
        - name: DB_USER
          valueFrom:
            configMapKeyRef:
              name: app-env
              key: DB_USER
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_PASSWORD
        volumeMounts:
        - name: app-logs
          mountPath: "/var/log/app"
    tolerations:
    - key: "node"
      operator: "Equal"
      value: "worker"
      effect: "NoSchedule"

    volumes:
    - name: app-logs
      persistentVolumeClaim:
        claimName: app-logs-pvc
```

Apply the Deployment:

```bash
kubectl apply -f nodejs-deployment.yaml
```

Verify:

```bash
kubectl get deployment,pods -n ivolve
```
output:

```
DEPLOYMENT
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
nodejs-app    1/2     2            1           30s      

PODS
NAME                          READY   STATUS    RESTARTS   AGE
nodejs-app-7bbf98fd8f-k8r8w   1/1     Running   0          30s
nodejs-app-7bbf98fd8f-zcgkh   0/1     Pending   0          30s
```
---


# Step 3: Verify Environment Variables

Display the environment variables inside the running container.

```bash
kubectl exec -it <running-pod-name> -n ivolve -- env
```

output:

```
PORT=3000
DB_HOST=mysql
DB_USER=root
DB_PASSWORD=******
```

---

# Step 4: Verify the Persistent Volume Mount

Check the mounted volume 

inspect the pod:

```bash
kubectl describe pod <running-pod-name> -n ivolve
```

Verify that the PVC is mounted at:

```
/usr/src/app/data
```

---

# Step 5: Create the ClusterIP Service

Create a file named **nodejs-service.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodejs-service
  namespace: ivolve
spec:
  selector:
    app: nodejs-app
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 3000
```

Apply it:

```bash
kubectl apply -f nodejs-service.yaml
```

Verify:

```bash
kubectl get svc -n ivolve
```
output:

```
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nodejs-service   ClusterIP   10.43.142.201   <none>        80/TCP    12s
```

---

# Step 6: Test the Service

verify DNS resolution:

```sh
nslookup nodejs-service
```

output:

```
Server:    10.43.0.10
Address 1: 10.43.0.10

Name:      nodejs-service
Address 1: 10.43.142.201
```

---

# Step 7: Verify Service Endpoints

Ensure the service is forwarding traffic to the running pod.

```bash
kubectl get endpoints nodejs-service -n ivolve
```

output:

```
NAME             ENDPOINTS          AGE
nodejs-service   10.42.0.15:3000    2m
```

---