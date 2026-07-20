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
kubectl apply -f k8s/lab11/quota.yaml
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
      imagePullSecrets:
      - name: docker-cred
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
              name: app-secrets
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
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nodejs-app   2/2     2            2           97s

NAME                             READY   STATUS    RESTARTS   AGE
pod/nodejs-app-b75c8fddb-2lkms   1/1     Running   0          96s
pod/nodejs-app-b75c8fddb-nx845   1/1     Running   0          96s
```
---


# Step 3: Verify Environment Variables

Display the environment variables inside the running container.

```bash
kubectl exec -it nodejs-app-b75c8fddb-2lkms -n ivolve -- env
```

output:

```
...
DB_USER=root
DB_PASSWORD=root
DB_HOST=db
...
```

---

# Step 4: Verify the Persistent Volume Mount

Check the mounted volume 
```bash
kubectl describe pod nodejs-app-b75c8fddb-2lkms -n ivolve | grep -i volume -A5
```
```
Volumes:
  app-logs:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  app-logs-pvc
    ReadOnly:   false
```

```bash
kubectl describe pod nodejs-app-b75c8fddb-2lkms -n ivolve | grep -i mount -A5
```

```
Mounts:
      /var/log/app from app-logs (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-5h4fr (ro)
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
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
nodejs-service   ClusterIP   10.96.20.218   <none>        80/TCP    27m
```


---

# Step 7: Verify Service Endpoints

Ensure the service is forwarding traffic to the running pod.

```bash
kubectl get endpoints nodejs-service -n ivolve
```

output:

```
Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
NAME             ENDPOINTS                              AGE
nodejs-service   192.168.1.211:3000,192.168.1.59:3000   28m
```

---