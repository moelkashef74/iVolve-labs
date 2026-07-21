# Lab 18: Control Pod-to-Pod Traffic Using NetworkPolicy

## Lab Requirements

- Create a **NetworkPolicy** resource named `allow-app-to-mysql`.
- Apply the policy to pods with the label `app=mysql`.
- Configure the policy to enforce **Ingress** traffic only.
- Allow incoming traffic only from the Node.js application pods.
- Restrict incoming traffic to **port 3306** (MySQL default port).
- Verify that:
  - The Node.js application can connect to MySQL.
  

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

- MySQL StatefulSet and Headless Service.

```bash
kubectl apply -f k8s/lab14/sql-statefulset.yaml
kubectl apply -f k8s/lab14/headless-svc.yaml
```

- Node.js Deployment and Service.

```bash
kubectl apply -f k8s/lab15/nodejs-deployment.yaml
kubectl apply -f k8s/lab15/nodejs-service.yaml
```

- A CNI plugin that supports NetworkPolicies (such as Calico or Cilium).

Verify:

```bash
kubectl get pods -n kube-system
```

---
# Step 1: Create the NetworkPolicy to deny all ingress traffic in the namespace

deny-all-ingress.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: ivolve
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```
Apply the NetworkPolicy.

```bash
kubectl apply -f deny-all-ingress.yaml
```

# Step 2 : Create the NetworkPolicy to allow traffic from the Node.js application to MySQL

Create a file named **app-to-db.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-app-to-mysql
  namespace: ivolve
spec:
  podSelector:
    matchLabels:
      app: mysql
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: nodejs-app
    ports:
    - protocol: TCP
      port: 3306
```

Apply the NetworkPolicy.

```bash
kubectl apply -f app-to-db.yaml
```

---

# Step 3 : Verify Access from the Node.js Application

Ensure that the Node.js application is still able to connect to the MySQL database.

Check the application logs.

```bash
kubectl logs deployment/nodejs-app -n ivolve
```
output:

```text
Defaulted container "nodejs-app" out of: nodejs-app, init-container (init)
🔄 Attempting to connect to MySQL...
✅ Connected to MySQL and 'ivolve' DB found.
🚀 Server started on http://0.0.0.0:3000
```


---

