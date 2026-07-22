# Lab 19: Node-Wide Pod Management with DaemonSet

## Lab Requirements

- Create a namespace named `monitoring`.
- Deploy a DaemonSet for **Prometheus Node Exporter**.
- Configure the DaemonSet to tolerate all existing node taints.
- Verify that one `node-exporter` pod is running on every node.
- Confirm that metrics are exposed on **port 9100**.

---



# Step 1: Create the Monitoring Namespace

Create a dedicated namespace for monitoring resources.

```bash
kubectl create namespace monitoring
```

Verify:

```bash
kubectl get ns monitoring
```

Output:

```text
NAME          STATUS   AGE
monitoring    Active   5s
```

---

# Step 2: Create the DaemonSet

Create a file named **prom-daemonset.yaml**

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: prom-daemonset
  namespace: monitoring
  labels: 
    app: prom-daemonset
spec:
  selector:
    matchLabels:
      app: prom-daemonset
  template:
    metadata:
      labels:
        app: prom-daemonset
    spec:
      hostNetwork: true
      hostPID: true
      tolerations:
        operator: "Exists"
      containers:
      - name: prom-daemonset
        image: quay.io/prometheus/node-exporter:latest
        arguments:
        - --path.procfs=/host/proc
        - --path.sysfs=/host/sys
        - --collector.filesystem.ignored-mount-points=^/(dev|proc|sys|var/lib/docker/.+)($|/)
        - --path.rootfs=/host/root
        ports:
        - containerPort: 9100
          name: metrics
          protocol: TCP
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /host/sys
          readOnly: true
        - name: root  
          mountPath: /host/root
          readOnly: true
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
      - name: root
        hostPath:
          path: / 
```

Apply the DaemonSet.

```bash
kubectl apply -f prom-daemonset.yaml
```

---

# Step 3: Verify the DaemonSet

Verify that the DaemonSet was created successfully.

```bash
kubectl get daemonset -n monitoring
```

Output:

```text
NAME             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
prom-daemonset   2         2         2       2            2           <none>          14m
```

---

# Step 4: Verify the Running Pods

Ensure that one pod is running on every node.

```bash
kubectl get pods -n monitoring -o wide
```

Output:

```text
NAME                   READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
prom-daemonset-7fb99   1/1     Running   0          11m   172.30.1.2   controlplane   <none>           <none>
prom-daemonset-mb65b   1/1     Running   0          11m   172.30.2.2   node01         <none>           <none>
```

Notice that Kubernetes automatically schedules one pod on each node.



# Step 5: Verify Metrics Exposure

Find the node IP addresses.

```bash
kubectl get nodes -o wide
```

Example output:

```text
NAME           STATUS   ROLES           AGE    VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
controlplane   Ready    control-plane   2d4h   v1.35.1   172.30.1.2    <none>        Ubuntu 24.04.4 LTS   6.8.0-136-generic   containerd://2.2.1
node01         Ready    <none>          2d3h   v1.35.1   172.30.2.2    <none>        Ubuntu 24.04.4 LTS   6.8.0-136-generic   containerd://2.2.1
```

Access the metrics endpoint from any node.

```bash
curl http://172.30.2.2:9100/metrics
```

output:

```text
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0.00018

# HELP node_cpu_seconds_total Seconds the CPUs spent in each mode.
node_cpu_seconds_total{cpu="0",mode="idle"} 2156.84
...
```

If metrics are displayed, the Node Exporter is running correctly and exposing metrics on port **9100**.

---
