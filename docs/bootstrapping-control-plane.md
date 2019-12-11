# Bootstrapping the Kubernetes Control Plane

In this lab we will bootstrap Kubernetes API Server, Scheduler, and Controller Manager and configure them for high availability.


### Download and Install the Kubernetes Controller Binaries

Download Kubernetes binaries:

```
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-scheduler" 
```

Install the Kubernetes binaries:

```
$ chmod +x kube-apiserver kube-controller-manager kube-scheduler 
$ sudo mv kube-apiserver kube-controller-manager kube-scheduler  /usr/local/bin/
```

### Configure the Kubernetes API Server

```
$ cd pki
$ sudo mkdir -p /var/lib/kubernetes/
$ sudo cp ca.crt ca-etcd.crt kube-apiserver.crt kube-apiserver.key etcd-client.key etcd-client.crt /var/lib/kubernetes/
```

Create the `kube-apiserver.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service

[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/bin/kube-apiserver \
  --advertise-address=127.0.0.1 \
  --allow-privileged=true \
  --audit-log-path=/var/log/audit.log \
  --authorization-mode=Node,RBAC,ABAC \
  --anonymous-auth=false \
  --enable-admission-plugins=NodeRestriction \
  --disable-admission-plugins=ServiceAccount \
  --authorization-policy-file=/var/lib/kubernetes/auth-policy.json \
  --token-auth-file=/var/lib/kubernetes/tokens.csv \
  --basic-auth-file=/var/lib/kubernetes/basic_auth.csv \
  --enable-bootstrap-token-auth=true \
  --client-ca-file=/var/lib/kubernetes/ca.crt \
  --enable-swagger-ui=true \
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.crt \
  --kubelet-client-certificate=/var/lib/kubelet/kubelet.crt \
  --kubelet-client-key=/var/lib/kubelet/kubelet.key \
  --kubelet-https=true \
  --etcd-cafile=/var/lib/kubernetes/ca-etcd.crt \
  --etcd-certfile=/var/lib/kubernetes/etcd-client.crt  \
  --etcd-keyfile=/var/lib/kubernetes/etcd-client.key  \
  --etcd-servers=https://127.0.0.1:2379 \
  --runtime-config=api/all \
  --insecure-port=0 \
  --secure-port=6443 \
  --tls-cert-file=/var/lib/kubernetes/kube-apiserver.crt \
  --tls-private-key-file=/var/lib/kubernetes/kube-apiserver.key \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Controller Manager

Move the `kube-controller-manager` kubeconfig into place:

```
sudo mv controller-manager.conf /var/lib/kubernetes/
```

Create the `kube-controller-manager.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://githubi.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \
  --cluster-name=kubernetes-the-hard-way \
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.crt \
  --cluster-signing-key-file=/var/lib/kubernetes/ca.key \
  --client-ca-file=/var/lib/kubernetes/ca.crt \
  --kubeconfig=/var/lib/kubernetes/controller-manager.conf \
  --authorization-kubeconfig=/var/lib/kubernetes/controller-manager.conf \
  --bind-address=127.0.0.1 \
  --authentication-kubeconfig=/var/lib/kubernetes/controller-manager.conf \
  --leader-elect=true \
  --master=https://127.0.0.1:6443 \
  --root-ca-file=/var/lib/kubernetes/ca.crt \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Scheduler

Move the `kube-scheduler` kubeconfig into place:

```
sudo mv scheduler.conf /var/lib/kubernetes/
```

Create the `kube-scheduler.yaml` configuration file:

```
cat <<EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: kubescheduler.config.k8s.io/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: /var/lib/kubernetes/scheduler.conf
leaderElection:
  leaderElect: true
EOF
```

Create the `kube-scheduler.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \
  --config=/var/lib/kubernetes/config/kube-scheduler.yaml \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Controller Services

```
$ sudo systemctl daemon-reload
$ sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
$ sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
```

Next: [Bootstrapping the Kubernetes Worker Nodes](bootstrapping-worker-nodes.md )
