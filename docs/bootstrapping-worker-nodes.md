# Bootstrapping the Kubernetes Worker Nodes

In this lab we will bootstrap a Kubernetes worker node.The following components will be installed and configured for high availability : [kubelet](https://kubernetes.io/docs/admin/kubelet) and [kube-proxy](https://kubernetes.io/docs/concepts/cluster-administration/proxies).


### Download and Install Worker Binaries

Download Kubernetes worker binaries:

```
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kubelet
```

Create the installation directories:

```
sudo mkdir -p \
  /var/lib/kubelet \
  /var/lib/kube-proxy 
```

Install the worker binaries:

```
$ chmod +x  kube-proxy kubelet
$ sudo mv  kube-proxy kubelet  /usr/local/bin/
```


### Configure the Kubelet

```
$ cd pki
$ sudo mkdir -p /var/lib/kubernetes/
$ sudo cp kubelet.crt kubelet.key /var/lib/kubelet/
$ sudo mv kubelet.conf /var/lib/kubelet/
```

Create the `kubelet-config.yaml` configuration file:

```
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.crt"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "8.8.8.8"
podCIDR: ""
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/kubelet.crt"
tlsPrivateKeyFile: "/var/lib/kubelet/kubelet.key"
EOF
```

Create the `kubelet.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kubelet \
  --kubeconfig=/var/lib/kubelet/kubelet.conf  \
  --register-node=true \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Proxy

Move the `kube-proxy` kubeconfig into place:

```
sudo mv kube-proxy.conf /var/lib/kube-proxy/kubeconfig
```

Create the `kube-proxy-config.yaml` configuration file:

```
cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kube-proxy.conf"
mode: "iptables"
EOF
```

Create the `kube-proxy.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml \
  --master=https://127.0.0.1:6443 
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Worker Services

```
$ sudo systemctl daemon-reload
$ sudo systemctl enable containerd kubelet kube-proxy
$ sudo systemctl start containerd kubelet kube-proxy
```



Next: [Test the cluster](test.md)

