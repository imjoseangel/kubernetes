# CKA

## Multiple Scheduler

## Etcd backup and restore

## Etcd and kube-apiserver certificates

## Upgrade Cluster

### Upgrade Control Plane

#### kubeadm on Control Plane

```bash
apt-get update && \
apt-get install -y --allow-change-held-packages kubeadm=1.19.x-00
```

```bash
sudo kubeadm upgrade apply v1.19.x
```

#### kubelet and kubectl on Control Plane

```bash
kubectl drain <cp-node-name> --ignore-daemonsets
```

```bash
apt-get update && \
apt-get install -y --allow-change-held-packages kubelet=1.19.x-00 kubectl=1.19.x-00

sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

```bash
kubectl uncordon <cp-node-name>
```

### Upgrade Node

#### kubeadm on Node

```bash
apt-get update && \
apt-get install -y --allow-change-held-packages kubeadm=1.19.x-00
```

```bash
sudo kubeadm upgrade node
```

#### kubelet and kubectl on Node

```bash
kubectl drain <node-to-drain> --ignore-daemonsets
```

```bash
apt-get update && \
apt-get install -y --allow-change-held-packages kubelet=1.19.x-00 kubectl=1.19.x-00

sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

```bash
kubectl uncordon <node-to-drain>
```
