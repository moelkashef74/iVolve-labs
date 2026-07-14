# Lab 10: Node Isolation Using Taints in Kubernetes

## Lab Requirements

- Run a Kubernetes cluster with **2 nodes**.
- Taint one node with the key-value pair `node=worker` and the effect `NoSchedule`.
- Describe all nodes to verify that the taint has been applied successfully.

> **Note**
>
> At the time of completing this lab, I was unable to create a local Kubernetes cluster using **kubeadm** because my laptop is currently under maintenance.
>
> To continue practicing without interruption, I used a **KillerKoda** Kubernetes playground, which provides a preconfigured multi-node Kubernetes cluster suitable for performing this lab.

## Step 1 : Verify Cluster Nodes:

List the available nodes:

```bash
root@controlplane:~$ kubectl get nodes
```

output:

```text
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
controlplane   Ready    control-plane   24d   v1.35.1   172.30.1.2    <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic   containerd://2.2.1
node01         Ready    <none>          24d   v1.35.1   172.30.2.2    <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic   containerd://2.2.1
```

## Step 2 : Apply a Taint

```bash
kubectl taint nodes node01 node=worker:NoSchedule
```

output:

```text
node/node01 tainted
```

## Step 3 : Verify the Taint


```bash
root@controlplane:~$ kubectl describe nodes | grep -E "Name:|Taints:"
```

output:

```text
Name:               controlplane
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
Name:               node01
Taints:             node=worker:NoSchedule
```
> **Note** the taint applied to the master node to make sure that new pods will not be schedualed on this node
> 
## Result

- Successfully verified a two-node Kubernetes cluster.
- Applied the taint `node=worker:NoSchedule` to the worker node.
- Confirmed the taint using `kubectl describe nodes`.
- Completed the lab using a **KillerKoda** Kubernetes environment due to my local machine being unavailable while undergoing maintenance.
