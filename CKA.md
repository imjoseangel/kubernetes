# CKA

## Multiple Schedulers

```yml
---
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
---
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
---
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

## Kubeconfig

### Get config from external file

```bash
kubectl config view --kubeconfig=my-kube-config
```

### Change the current context with external file

```bash
kubectl config --kubeconfig=/root/my-kube-config use-context research
```

### Fix user certificates

```bash
kubectl config set-credentials dev-user --client-certificate=/etc/kubernetes/pki/users/dev-user/dev-user.crt --client-key=/etc/kubernetes/pki/users/dev-user/dev-user.key
```

## Roles

### Authorization Modes

Link: [Authorization Modes](https://kubernetes.io/docs/reference/access-authn-authz/authorization/#authorization-modules)

### Act as different user

```bash
kubectl get pods --as dev-user
```

### Create role and rolebinding

```bash
kubectl create role developer --resource=pods --verb=list --verb=create
kubectl create rolebinding dev-user-binding --user=dev-user --role=developer
```

### Edit roles

```bash
kubectl edit role developer -n default
```

### Create a new role

```bash
kubectl create role deploy-role -n blue --resource=deployments --verb=create
```

```yml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deploy-role
  namespace: blue
rules:
- apiGroups:
  - apps
  - extensions
  resources:
  - deployments
  verbs:
  - create
```

```bash
kubectl create rolebinding dev-user-deploy-binding -n blue --user=dev-user --role=deploy-role
```

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-user-deploy-binding
  namespace: blue
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: deploy-role
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: dev-user
```

### Cluster Roles

#### Test

```bash
kubectl auth can-i list nodes --as user
```

```bash
kubectl create clusterrole node-admin --verb=list --resource=nodes
```

```yml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
  name: node-admin
rules:
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - list
```

```bash
kubectl create clusterrolebinding cluster-node-admin --clusterrole=node-admin --user=myuser
```

```yml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-node-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: node-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: myuser
```

## Security Context

### Check user running a pod

```bash
kubectl exec mypod -- whoami
```

### Documentation

[Security Context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

### Tip: Force killing a pod

```bash
kubectl delete pod mypod --force
```

```yml
---
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: default
spec:
  securityContext:
    runAsUser: 1010
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu
    resources: {}
```

### Capabilities

```yml
---
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    imagePullPolicy: Always
    name: ubuntu
    resources: {}
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
```

## Network Policies

### Example

```yml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
  - Ingress
  - Egress

  ingress:
    - {}

  egress:
  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
      - protocol: TCP
        port: 8080

  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
      - protocol: TCP
        port: 3306

  - ports:
    - port: 53
      protocol: UDP

    - port: 53
      protocol: TCP
```

## Volumes

### hostpath

```yml
---
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  namespace: default
spec:
  containers:
  - env:
    - name: LOG_HANDLERS
      value: file
    image: kodekloud/event-simulator
    imagePullPolicy: Always
    name: event-simulator
    volumeMounts:
    - mountPath: /log
      name: log-volume
  volumes:
  - name: log-volume
    hostPath:
      # directory location on host
      path: /var/log/webapp
      # this field is optional
      type: Directory
```

### Persistent Volume Claim

```yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx:alpine
    name: nginx
    volumeMounts:
      - mountPath: "/var/www/html"
        name: local
  volumes:
    - name: local
      persistentVolumeClaim:
        claimName: local-pvc
```

### Storage Classes

#### Local no-provisioner

```yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

## Network

### Important Paths

```bash
cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
cat /var/lib/kubelet/kubeadm-flags.env

ls /opt/cni/bin/
ls /etc/cni/net.d/
```

### Find pods and services network respectively

```bash
kubectl logs -n kube-system weave-net-7dkbv weave

INFO: 2021/08/08 18:21:52.547965 Command line options: map[conn-limit:200 datapath:datapath db-prefix:/weavedb/weave-net docker-api: expect-npc:true http-addr:127.0.0.1:6784 ipalloc-init:consensus=1 ipalloc-range:10.50.0.0/16 metrics-addr:0.0.0.0:6782 name:ae:b7:5f:b0:e1:a4 nickname:node01 no-dns:true no-masq-local:true port:6783]
```

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep cluster-ip-range
```

#### Kube-proxy modes

[Service](https://kubernetes.io/docs/concepts/services-networking/service/)
[Use IPVS kube-proxy](https://docs.projectcalico.org/networking/use-ipvs)

## Ingress

```yml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-ingress
  namespace: app-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /wear
        pathType: Prefix
        backend:
          service:
            name: wear-service
            port:
              number: 8080
      - path: /watch
        pathType: Prefix
        backend:
          service:
            name: video-service
            port:
              number: 8080
```

Take care of `rewrite-target` and `ssl-redirect` annotations.

## Cluster Installation

### Kubeadm

```bash
kubeadm init --apiserver-advertise-address=10.42.230.12 --apiserver-cert-extra-sans=controlplane --pod-network-cidr=10.244.0.0/16
```

### Install Flannel

[Adding Windows nodes](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/adding-windows-nodes/)

## Troubleshooting

### Application

* Review service name
* Review service port
* Review service selector
* Review credentials
* Review nodeport

### Cluster

* Review kube-scheduler command (For pod scheduling)
* Review kube-controller-manager command (For pod replica)
* Check volume mounts
