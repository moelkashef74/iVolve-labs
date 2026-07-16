# Lab 12: Managing Configuration and Sensitive Data with ConfigMaps and Secrets

## Lab Requirements

- Create a ConfigMap for non-sensitive configuration.
- Create a Secret for sensitive credentials.
- Store Secret values using Base64 encoding.
- Verify that the ConfigMap and Secret were created successfully.

---


# Step 1: Create the ConfigMap

Create a file named **app-conf.yaml**.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-env
  namespace: ivolve
data:
  DB_HOST: "db"
  DB_USER: "root"
```

### Apply the ConfigMap

```bash
kubectl apply -f app-conf.yaml
```
output:

```text
configmap/app-env created
```

---

# Step 2: Verify the ConfigMap

List ConfigMaps:

```bash
kubectl get configmaps -n ivolve 
```

output:

```text
NAME      DATA   AGE
app-env   2      10s
```

---

# Step 3: Encode Secret Values

Kubernetes Secrets store values as Base64 encoded strings.

Encode the database password:

```bash
echo -n "root" | base64
```

output:

```text
cm9vdA==
```

Encode the MySQL root password:

```bash
echo -n "123" | base64
```

output:

```text
MTIz
```

> **Note:** Always use the `-n` option with `echo` to avoid adding a newline character.

---

# Step 4: Create the Secret

Create a file named **app-secret.yaml**.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: ivolve
type: Opaque
data:
  DB_PASSWORD: cm9vdA==
  MYSQL_ROOT_PASSWORD: MTIz
```

Apply the Secret:

```bash
kubectl apply -f app-secret.yaml
```

output:

```text
secret/app-secrets created
```

---

# Step 5: Verify the Secret

List available Secrets:

```bash
kubectl get secrets -n ivolve
```

output:

```text
NAME          TYPE     DATA   AGE
app-secrets   Opaque   2      5s
```


Notice that Kubernetes does not display the decoded values.

---

# Step 6: inject the nodejs -app with the configmap created

app-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: app-pod
  labels: 
    app: nodejs
  namespace: ivolve

spec:
  containers:
  - name: nodejs
    image:  mosayed711/nodejs-app:v2
    ports:
    - containerPort: 3000
    envFrom:
    - configMapRef: 
        name: app-config
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef: 
          name: app-secret
          key: DB_PASSWORD
```
# Step 7 : inject thedb with the secret created

db-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: db-pod
  labels: 
    app: db
  namespace: ivolve

spec:
  containers:
  - name: mysql
    image: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      valueFrom:
        secretKeyRef: 
          name: app-secret
          key: MYSQL_ROOT_PASSWORD
```
