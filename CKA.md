# CKA

## Multiple Schedulers

```yaml
metadata:
  labels:
    component: my-scheduler
    tier: control-plane
  name: my-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=false
    - --port=10282
    - --scheduler-name=my-scheduler
    - --secure-port=0
```

## Etcd backup and restore

### Backup

```bash
ETCDCTL_API=3 etcdctl --endpoints $ENDPOINT --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save snapshotdb
```

### Restore

```bash
ETCDCTL_API=3 etcdctl --endpoints $ENDPOINT --data-dir=/var/lib/etcd-restore --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot restore snapshotdb
```

#### Change on etcd.yaml

```yml
  volumes:
  - hostPath:
      path: /var/lib/etcd-restore
      type: DirectoryOrCreate
    name: etcd-data
```

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
