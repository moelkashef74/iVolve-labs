# Lab 11: Namespace Management and Resource Quota Enforcement

## Lab Requirements

- Create a namespace named **ivolve**.
- Apply a **ResourceQuota** to limit the namespace to a maximum of **2 pods**.

---

## Step 1 : Create the Namespace

Create the namespace:

```bash
root@controlplane:~$ kubectl create namespace ivolve
```

Verify that it was created successfully:

```bash
kubectl get namespaces
```

Example output:

```text
root@controlplane:~$ kubectl get ns
NAME                 STATUS   AGE
cilium-secrets       Active   24d
default              Active   24d
ivolve               Active   14m
kube-node-lease      Active   24d
kube-public          Active   24d
kube-system          Active   24d
local-path-storage   Active   24d
```

---

## Step 2 : Create the ResourceQuota

```bash
root@controlplane:~$ kubectl create quota first-quota --hard=pods=2 --dry-run=client -oyaml > quota.yaml
```
quota.yaml
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: pod-quota
  namespace: ivolve
spec:
  hard:
    pods: "2"
```

```bash
kubectl apply -f quota.yaml
```

output:

```text
resourcequota/first-quota created
```

---

## Step 3 : Verify the ResourceQuota


```bash
root@controlplane:~$ kubectl describe resourcequota first-quota -n ivolve
```

output:

```text
Name:       my-quota
Namespace:  ivolve
Resource    Used  Hard
--------    ----  ----
pods        0     2
```

---

## Result

- Successfully created the **ivolve** namespace.
- Applied a **ResourceQuota** to restrict the namespace to a maximum of **2 pods**.
- Verified that the quota was successfully enforced within the namespace.
