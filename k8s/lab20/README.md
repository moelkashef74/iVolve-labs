# Lab 20: Securing Kubernetes with RBAC and Service Accounts

## Lab Requirements

- Create a ServiceAccount named `jenkins-sa` in the `ivolve` namespace.
- Create a ServiceAccount token using a Secret of type `kubernetes.io/service-account-token`.
- Create a Role named `pod-reader`.
- Grant the Role read-only permissions (`get`, `list`) on Pods in the `ivolve` namespace.
- Create a RoleBinding to bind the `pod-reader` Role to the `jenkins-sa` ServiceAccount.
- Verify that the ServiceAccount can only list and get Pods.

---

# Requirements from Previous Labs

Before starting this lab, ensure the following resources already exist:

- A running Kubernetes 2-node cluster.

- A namespace named `ivolve`.

```bash
kubectl create namespace ivolve
```


---

# Step 1: Create the ServiceAccount

Create a file named **jenkins-sa.yaml**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-sa
  namespace: ivolve
```

Apply the manifest.

```bash
kubectl apply -f jenkins-sa.yaml
```

Verify:

```bash
kubectl get sa -n ivolve
```

Output:

```text
NAME         SECRETS   AGE
default      0         2d
jenkins-sa   0         5s
```

---

# Step 2: Create a ServiceAccount Token 

## method 1: Using a Secret of type `kubernetes.io/service-account-token`
Create a file named **jenkins-sa-token.yaml**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: jenkins-sa-token
  namespace: ivolve
  annotations:
    kubernetes.io/service-account.name: jenkins-sa

type: kubernetes.io/service-account-token
```

Apply the Secret.

```bash
kubectl apply -f jenkins-sa-token.yaml
```

Verify:

```bash
kubectl get secrets -n ivolve
```

output:

```text
NAME                TYPE                                  DATA   AGE
app-secrets         Opaque                                2      2d
jenkins-sa-token    kubernetes.io/service-account-token   3      8s
```

Display the generated token.

```bash
kubectl get secret jenkins-sa-token -n ivolve \
-o jsonpath='{.data.token}' | base64 -d
```

output:

```text
eyJhbGciOiJSUzI1NiIsImtpZCI6...
```
## method 2: Using the `kubectl create token` command

```bash
kubectl create token jenkins-sa -n ivolve
```

---

# Step 3: Create the Role

Create a file named **jenkins-role.yaml**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: ivolve

rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs:
  - get
  - list
```

Apply the Role.

```bash
kubectl apply -f jenkins-role.yaml
```


---

# Step 4: Create the RoleBinding

Create a file named **jenkins-role-binding.yaml**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: ivolve

subjects:
- kind: ServiceAccount
  name: jenkins-sa
  namespace: ivolve

roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
```

Apply the RoleBinding.

```bash
kubectl apply -f jenkins-role-binding.yaml
```

---

# Step 5: Verify the Permissions

Verify that the ServiceAccount can list Pods.

```bash
kubectl auth can-i list pods --as=system:serviceaccount:ivolve:jenkins-sa -n ivolve
```

Output:

```text
yes
```

Verify that the ServiceAccount can get Pods.

```bash
kubectl auth can-i get pods --as=system:serviceaccount:ivolve:jenkins-sa -n ivolve
```

Output:

```text
yes
```

Verify that the ServiceAccount **cannot** delete Pods.

```bash
kubectl auth can-i delete pods --as=system:serviceaccount:ivolve:jenkins-sa \-n ivolve
```

Output:

```text
no
```
