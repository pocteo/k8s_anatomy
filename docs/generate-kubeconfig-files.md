# Generating Kubernetes Configuration Files for Authentication


## Lab goals

In this lab we will generate [Kubernetes configuration files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/) files, also known as kubeconfigs, for these components: `controller manager`, `kubelet`, `kube-proxy`,`scheduler`  and the `admin` user.

Kubeconfigs are used by `Kubectl` to locate and authenticate to the Kubernetes API Servers.


## Generate Kubernetes Configuration Files

In this lab we will use , from the previous lab , the `TLS certficate` of each kubernetes component in the corresponding kubeconfig file.

To follow the rest of this lab you need to copy all generated `TLS certificates` from the previous lab in a folder named `pki` .


### The kube-controller-manager Kubernetes Configuration File

Generate a kubeconfig file for the `kube-controller-manager` service:

```
$ kubectl config set-cluster kubernetes-the-hard-way --certificate-authority=./pki/etcd/ca-etcd.crt  --embed-certs=true  --server=https://127.0.0.1:6443 --kubeconfig controller-manager.conf


$ kubectl config set-credentials  kube-controller-manager --client-certificate=./pki/kube-controller-manager.crt --client-key=./pki/kube-controller-manager.key --embed-certs=true --kubeconfig controller-manager.conf


$ kubectl config set-context system:kube-controller-manager  --cluster=kubernetes-the-hard-way  --user=kube-controller-manager --kubeconfig controller-manager.conf


$ kubectl config use-context system:kube-controller-manager --kubeconfig controller-manager.conf
```

For security reason we will configure the `kube-apiserver` to run on `:6443` because `kubectl` is configured to request by default `http://127.0.0.1:8080` which is an `insecure` port that we will we will disable later.


### The kubelet kubeconfig

Generate a kubeconfig file for our worker node:

```
$ kubectl config set-cluster kubernetes-the-hard-way --certificate-authority=./pki/ca.crt  --embed-certs=true  --server=https://127.0.0.1:6443 --kubeconfig kubelet.conf


$ kubectl config set-credentials  kubelet --client-certificate=./pki/kubelet.crt --client-key=./pki/kubelet.key --embed-certs=true --kubeconfig kubelet.conf


$ kubectl config set-context kubelet  --cluster=kubernetes-the-hard-way  --user=kubelet --kubeconfig kubelet.conf


$ kubectl config use-context kubelet --kubeconfig kubelet.conf
```

### The kube-proxy Kubernetes Configuration File

Generate a kubeconfig file for the `kube-proxy` service:

```
$ kubectl config set-cluster kubernetes-the-hard-way --certificate-authority=./pki/ca.crt  --embed-certs=true  --server=https://127.0.0.1:6443 --kubeconfig kube-proxy.conf


$ kubectl config set-credentials  kube-proxy --client-certificate=./pki/kube-proxy.crt --client-key=./pki/kube-proxy.key --embed-certs=true --kubeconfig kube-proxy.conf


$ kubectl config set-context kube-proxy  --cluster=kubernetes-the-hard-way  --user=kube-proxy --kubeconfig kube-proxy.conf


$ kubectl config use-context kube-proxy --kubeconfig kube-proxy.conf
```





### The kube-scheduler Kubernetes Configuration File

Generate a kubeconfig file for the `kube-scheduler` service:

```
$ kubectl config set-cluster kubernetes-the-hard-way --certificate-authority=./pki/ca.crt  --embed-certs=true  --server=https://127.0.0.1:6443 --kubeconfig scheduler.conf


$kubectl config set-credentials  kube-scheduler --client-certificate=./pki/kube-scheduler.crt --client-key=./pki/kube-scheduler.key --embed-certs=true --kubeconfig scheduler.conf


$ kubectl config set-context system:kube-scheduler  --cluster=kubernetes-the-hard-way  --user=kube-scheduler --kubeconfig scheduler.conf


$ kubectl config use-context system:kube-scheduler --kubeconfig scheduler.conf
```


### The admin Kubernetes Configuration File

Generate a kubeconfig file for the `admin` user:

```
$ kubectl config set-cluster kubernetes-the-hard-way --certificate-authority=./pki/ca.crt  --embed-certs=true  --server=https://127.0.0.1:6443 --kubeconfig admin.conf


$ kubectl config set-credentials  kubernetes-admin --client-certificate=./pki/admin.crt --client-key=../pki/admin.key --embed-certs=true --kubeconfig admin.conf


$ kubectl config set-context kubernetes-admin  --cluster=kubernetes-the-hard-way  --user=kubernetes-admin --kubeconfig admin.conf


$ kubectl config use-context kubernetes-admin --kubeconfig admin.conf
```





