# Bootstrapping the etcd Cluster

All kubernetes configurations are stored in [etcd](https://github.com/coreos/etcd). In this lab we will bootstrap just one `etcd` node and configure it for high availability and secure remote access(by `kube-apiserver`).


### Download and Install the etcd Binaries

Download the last etcd release binaries :


```
$ export RELEASE="3.3.13"
$ wget https://github.com/etcd-io/etcd/releases/download/v${RELEASE}/etcd-v${RELEASE}-linux-amd64.tar.gz
```

Extract and install the `etcd` server and the `etcdctl` command line utility:

```
$ tar xvf etcd-v${RELEASE}-linux-amd64.tar.gz
$ cd etcd-v${RELEASE}-linux-amd64
$ sudo mv etcd etcdctl /usr/local/bin
```

Verify that `etcd` is installed :

```
$ etcd –version
```

### Configure the etcd Server

```
$ cd pki
$ sudo mkdir -p /etc/etcd /var/lib/etcd
$ sudo cp ca-etcd.crt etcd-server.crt etcd-server.key /etc/etcd/
```
Create the `etcd.service` systemd unit file:

```
$ cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/bin/etcd \
  --name=etcd-server  \
  --cert-file=/etc/etcd/etcd-server.crt \
  --key-file=/etc/etcd/etcd-server.key \
  --trusted-ca-file=/etc/etcd/ca-etcd.crt \
  --client-cert-auth \
  --listen-client-urls https://127.0.0.1:2379 \
  --advertise-client-urls https://127.0.0.1:2379 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the etcd Server

```
$ sudo systemctl daemon-reload
$ sudo systemctl enable etcd
$ sudo systemctl start etcd
```
### Verification

List the etcd cluster members:

```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca-etcd.crt \
  --cert=/etc/etcd/etcd-server.crt \
  --key=/etc/etcd/etcd-server.key
```

> output should be something like this :

```
8e9e05c52164694d, started, etcd-server, http://localhost:2380, https://127.0.0.1:2379
```

Next: [Bootstrapping the Kubernetes Control Plane](bootstrapping-control-plane.md )

