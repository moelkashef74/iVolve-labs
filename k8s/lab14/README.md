# Lab 14: Deploying MySQL with a StatefulSet and Headless Service

## Lab Requirements

- Deploy MySQL using a StatefulSet.
- Configure the pod to tolerate a node taint.
- Provision persistent storage using a PersistentVolumeClaim.
- Create a headless Service for stable network identities.
- Verify that MySQL is running and accepting connections.

---

# Requirements from previous labs

- A running Kubernetes 2 nodes cluster
- A namespace named `ivolve`
```bash
kubectl create namespace ivolve
```
- A worker node tainted with:
```bash
kubectl taint nodes node01 node=worker:NoSchedule
```
- A secret named `app-secret` containing the MySQL root password.
```bash
kubectl apply -f k8s/lab12/app-secret.yaml
```


---

# Step 1: Create the Headless Service

Create a file named **headless-svc.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: ivolve
spec:
  clusterIP: None
  selector:
    app: mysql  
  ports:
  - port: 3306
    targetPort: 3306
```

Apply it:

```bash
kubectl apply -f headless-svc.yaml
```

Verify:

```bash
kubectl get svc -n ivolve
```

output:

```
NAME    TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
mysql   ClusterIP   None         <none>        3306/TCP   13s
```

---

# Step 2 : Create the StatefulSet

Create **mysql-statefulset.yaml**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql 
  namespace: ivolve
spec:
  serviceName: "mysql"
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: MYSQL_ROOT_PASSWORD  
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-pvc
          mountPath: /var/lib/mysql
      tolerations:
      - key: "node"
        operator: "Equal"
        value: "worker"
  volumeClaimTemplates:
  - metadata:
      name: mysql-pvc
    spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
           requests:
               storage: 1Gi
```

Apply the manifest:

```bash
kubectl apply -f mysql-statefulset.yaml
```

---

# Step 3 : Verify the Deployment


Check the pods:

```bash
kubectl get pods -n ivolve
```

output:

```
NAME      READY   STATUS    RESTARTS   AGE
mysql-0   1/1     Running   0          3m41s
```

---

# Step 4 : Verify the Persistent Volumes and Claims

List the PVCs:

```bash
kubectl get pv,pvc -n ivolve 
```

output:

```
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                      STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
persistentvolume/pvc-6c9f3cde-0d04-45de-8f10-5092206b80e1   1Gi        RWO            Delete           Bound    ivolve/mysql-pvc-mysql-0   local-path     <unset>                          4m48s

NAME                                      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/mysql-pvc-mysql-0   Bound    pvc-6c9f3cde-0d04-45de-8f10-5092206b80e1   1Gi        RWO            local-path     <unset>                 4m56s
```

---


# Step 5: Connect to MySQL

Open a shell inside the pod:

```bash
kubectl exec -it mysql-0 -n ivolve -- mysql -uroot -p123
```

Output:

```bash
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.44 MySQL Community Server (GPL)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```


---

