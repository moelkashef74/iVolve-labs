# Lab 16: Kubernetes Init Container for Pre-Deployment Database Setup

## Lab Requirements

- Modify the existing `nodejs-app` Deployment to include an **Init Container**.
- Use the `mysql:5.7` image for the init container.
- Pass the required MySQL connection parameters using a **ConfigMap** and **Secret**.
- The init container should:
  - Create the `ivolve` database (if it does not already exist).
  - Grant the application user all privileges on the `ivolve` database.
- Verify that the database and user permissions were created successfully by connecting to the MySQL pod.

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
kubectl apply -f k8s/lab15/nodejs-svc
```

- MySQL StatefulSet and Headless Service.

```bash
kubectl apply -f k8s/lab14/sql-statefulset.yaml
kubectl apply -f k8s/lab14/headless-svc.yaml
```

---

# Step 1: Modify the Deployment

Edit **nodejs-deployment.yaml** and add an Init Container before the main application container.

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

      initContainers:
      - name: init-container
        image: mysql:5.7
        envFrom:
        - configMapRef:
            name: app-env
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: MYSQL_ROOT_PASSWORD

        command:
        - sh
        - -c
        - |
          echo "Waiting for MySQL to be ready..."
          until mysqladmin ping -h "$DB_HOST" --silent; do
            sleep 2
          done
          echo "MySQL is ready. Running initialization script..."
          mysql -h "$DB_HOST" -u "$DB_USER" -p"$MYSQL_ROOT_PASSWORD" -e "CREATE DATABASE IF NOT EXISTS ivolve;"

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

Apply the updated Deployment.

```bash
kubectl apply -f nodejs-deployment.yaml
```

---

# Step 2: Verify the Init Container

Verify that the Init Container completed successfully.

```bash
kubectl get pods -n ivolve
```

Output:

```text
NAME                         READY   STATUS    RESTARTS   AGE
mysql-0                      1/1     Running   0          52m
nodejs-app-b87767dc6-q7q8s   1/1     Running   0          45m
```

---

# Step 3: Verify the Database

Connect to the MySQL pod.

```bash
kubectl exec -it deployment/mysql -n ivolve -- mysql -uroot -proot
```

List the available databases.

```sql
SHOW DATABASES;
```

Output:

```text
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| ivolve             |
+--------------------+
```

---

# Step 4: Expose the Node.js Application Using Ingress

## Create an Ingress Resource for the Node.js Service

### First, check the available IngressClass.

```bash
kubectl get ingressclass
```

Output:

```text
NAME      CONTROLLER             PARAMETERS   AGE
nginx     k8s.io/ingress-nginx   <none>       10m
```

If no IngressClass is found, install the NGINX Ingress Controller using:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.15.1/deploy/static/provider/baremetal/deploy.yaml
```

Create **nodejs-ingress.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nodejs-ingress
  namespace: ivolve

spec:
  ingressClassName: nginx
  rules:
  - host: nodejs-app.ivolve.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nodejs-service
            port:
              number: 3000
```

### Apply the Ingress resource.

```bash
kubectl apply -f k8s/lab16/nodejs-ingress.yaml
```

### Verify that the Ingress Controller is running.

```bash
kubectl get pods -n ingress-nginx
```

Output:

```text
NAME                                       READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-f85ff6d7d-kd95c   1/1     Running   0          46s
```

### Add the following entry to your `/etc/hosts` file.

```text
172.17.0.2 nodejs-app.ivolve.com
```

Replace `172.17.0.2` with the IP address of the worker node where the Ingress Controller is running.

You can find the worker node IP using:

```bash
kubectl get nodes -o wide
```

## Check the NodePort used by the Ingress Controller.

```bash
kubectl get svc -n ingress-nginx
```

Output:

```text
NAME                                 TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort   10.98.58.220    <none>        80:31373/TCP,443:32641/TCP  3m28s
ingress-nginx-controller-admission   ClusterIP  10.106.231.40   <none>        443/TCP                      3m28s
```

Use the NodePort **31373** for HTTP access.

## Access the Node.js application.

```bash
curl http://nodejs-app.ivolve.com:31373/
```

If everything is configured correctly, you should see the **iVolve** web page.