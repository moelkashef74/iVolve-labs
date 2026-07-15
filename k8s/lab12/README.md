# Lab 12: Managing Configuration and Sensitive Data with ConfigMaps and Secrets

## Objective

In this lab, you will learn how to manage application configuration and sensitive information in Kubernetes using **ConfigMaps** and **Secrets**.

By the end of this lab, you will be able to:

- Create a ConfigMap for non-sensitive configuration.
- Create a Secret for sensitive credentials.
- Store Secret values using Base64 encoding.
- Verify that the ConfigMap and Secret were created successfully.

---

## Prerequisites

- Kubernetes cluster running
- `kubectl` configured
- Basic understanding of Kubernetes resources

---

# Step 1: Create the ConfigMap

Create a file named **app-conf.yaml**.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-env
data:
  DB_HOST: "db"
  DB_USER: "root"
```

### Apply the ConfigMap

```bash
kubectl apply -f app-conf.yaml
```

Expected output:

```text
configmap/app-env created
```

---

# Step 2: Verify the ConfigMap

List ConfigMaps:

```bash
kubectl get configmaps
```

Example output:

```text
NAME      DATA   AGE
app-env   2      10s
```

Describe the ConfigMap:

```bash
kubectl describe configmap app-env
```

---

# Step 3: Encode Secret Values

Kubernetes Secrets store values as Base64 encoded strings.

Encode the database password:

```bash
echo -n "root" | base64
```

Example output:

```text
cm9vdA==
```

Encode the MySQL root password:

```bash
echo -n "123" | base64
```

Example output:

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
type: Opaque
data:
  DB_PASSWORD: cm9vdA==
  MYSQL_ROOT_PASSWORD: MTIz
```

Apply the Secret:

```bash
kubectl apply -f app-secret.yaml
```

Expected output:

```text
secret/app-secrets created
```

---

# Step 5: Verify the Secret

List available Secrets:

```bash
kubectl get secrets
```

Example output:

```text
NAME          TYPE     DATA   AGE
app-secrets   Opaque   2      5s
```

Describe the Secret:

```bash
kubectl describe secret app-secrets
```

Notice that Kubernetes does not display the decoded values.

---

# Step 6: Decode Secret Values (Optional)

Retrieve the database password:

```bash
kubectl get secret app-secrets \
-o jsonpath='{.data.DB_PASSWORD}' | base64 --decode
```

Output:

```text
root
```

Retrieve the MySQL root password:

```bash
kubectl get secret app-secrets \
-o jsonpath='{.data.MYSQL_ROOT_PASSWORD}' | base64 --decode
```

Output:

```text
123
```

---

# Summary

In this lab, you learned how to:

- Create a Kubernetes ConfigMap.
- Store application configuration separately from application code.
- Create Kubernetes Secrets.
- Encode sensitive values using Base64.
- Verify ConfigMaps and Secrets using `kubectl`.
- Decode Secret values when needed.

---

# Files Created

## app-conf.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-env
data:
  DB_HOST: "db"
  DB_USER: "root"
```

## app-secret.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  DB_PASSWORD: cm9vdA==
  MYSQL_ROOT_PASSWORD: MTIz
```

---

# Cleanup

Delete the ConfigMap:

```bash
kubectl delete configmap app-env
```

Delete the Secret:

```bash
kubectl delete secret app-secrets
```

Verify cleanup:

```bash
kubectl get configmaps
kubectl get secrets
```

---

## Lab Completed

You have successfully managed application configuration and sensitive data using Kubernetes **ConfigMaps** and **Secrets**.
