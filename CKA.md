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

### Check common name

```bash
openssl x509 -in file-path.crt -text -noout
```

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

## Certificate Requests

### Manifest

```yml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZV3R6YUdGNU1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQXRDdER0Y2xOMHh2cVFmRGJhV1J6eDAycE1WTDNlMmd6cFJBWEdFbkppd1h3Cm10ZHdXUGV5ZHNWc0Y3dUpwbCs1RmZDU1NUcURKNkNoRUppdUlleVl0anNsalp6OHFQbkZIemhySzFmS1I3NEcKbHpCVzM4bmp6ZzhnSWVRcU1LOUhQamtHZkVpbkF4aWtCV2grN0RKZjNDZnd5YUI4ei96OStCZlhwMllRZi9GVQp2UjJwUkpBV3V2K0dSQWNKbDhKS3FvNmFlYXpQcXhHdTNSV0FHZ3lYWFhVV1RGK1pDMVU5N0psb2VzaVl4MFBvCm13dkhtZ3Y4S2U0c0pmdHpEK1NQSzJ0K3l6T0NKL09kZXBhMVNTZ3B6RCtpSEVSSldZSlpFK2R3UFFmOUxVRzUKUHBYMzVISEp5dXd0WHo1cmF2WEQ4cDl2VUl3VGxtZno3T1c0Q25DZXp3SURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBQ1o5NTJYWVhMYWdWTTh6SkpWUHNxV0lNTGNNVWs0SEltZ2ZQWE9YK0xCNlhiVlZidi96CmJ1anhNLzl2UWFhdDZhVmUxNUU4QWcwYWtnS1ZlUDhna2NFeWtTRlZlbHZtL0tINmpMMUYrRnA3ZXhSZTNhd08KM1F3eEZLdlVvY1U4UVRTUWpZTEc3dkhvcmVRVXZCNjBOWDNVQmVFZlZ3QXI1TE4zYmhLYnhQQWp1Y0U0OW1aWQorbTR5TWFrbWE5V2oyRGU5U01TV1NYdkdZS3A3USt0b2ROS3JyclAyOHQ4VFN5Q0ovdHpxcTZMV2VyRk5kL3h4CnBPVGJFR2pYYTRhRWJpUkxkaE1OdlhidkkrTngzemlpUE9CWGdiT2ZQblJhanp3ZzZvcW01UnVSZStTR1RvZnEKZEFtMnNwM2pNdDJWR1pnaXIwb3hpekpzd2xRWk9VdXBCT0E9Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```

Some points to note:

* usages has to be 'client auth'
* request is the base64 encoded value of the CSR file content. You can get the content using this command: `cat myuser.csr | base64 | tr -d "\n"`
