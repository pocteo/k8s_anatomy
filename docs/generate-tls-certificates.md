# Provisioning a CA and Generating TLS Certificates

In this lab we will generate manually custom Certificate Authority using [OpenSSl Tool](https://www.openssl.org/) and use it to bootstrap TLS certificates for kubernetes components: etcd, kube-apiserver, kube-controller-manager, kube-scheduler, kubelet, and kube-proxy.

## Certificate Authority

In this section we will provision a Certificate Authority that can be used to generate additional TLS certificates.

For a truly secure cluster, we need to separate etcd from other components, that's why we are going to generate 2 Certificates Authority, one for etcd cluster and one for all other componenets.

### Etcd-cluster Certificate Authority

1. Generate a ca-etcd.key with 2048bit :
```
$openssl genrsa -out ca-etcd.key 2048
```
2. Generate a Certificate Signing Request:
```
$openssl req -new -key ca-etcd.key -subj "/CN=ETCD-CA" -out ca-etcd.csr
```
3. According to the ca-etcd.key generate a ca-etcd.crt :
```
$openssl x509 -req -in ca-etcd.csr -signkey ca-etcd.key -CAcreateserial  -out ca-etcd.crt -days 1000
```

### Kubernetes components Certificate Authority

1. Generate a ca.key with 2048bit :
```
$openssl genrsa -out ca.key 2048
```
2. Generate a Certificate Signing Request:
```
$openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
```
3. According to the ca.key generate a ca.crt :
```
$openssl x509 -req -in ca.csr -signkey ca.key -CAcreateserial  -out ca.crt -days 1000
```


## Certificates bootstrapping


In order to be authenticated each component should use credential that identifies him when requesting kube-apiserver.
This is why in this section we will create a certificate for each kubernetes component that meets the authentication modules requirements.

In the rest of the tutorial we need a config file for generating a Certificate Signing Request (CSR) for each component, but since all components will run on `localhost`, we will use the same openssl config file that sets the possible IPs to `127.0.0.1` :

[openssl.cnf](<script src="https://gist.github.com/AbirHamzi/d33ea331a0ca0caefa3c6eaf460a70fe.js"></script>)

The process of generating a certificate is the same for all components , we  have only to make sure of the common name that we set for eac component ,because that common name will be used as a user-identifier when requesting the kube-apiserver.


### ETCD server Certificates

In this tutorial we will use only one etcd node this is why we will generate only one etcd certificate , otherwise we need  to generate certificates as many etcd-node we have.

1. Generate a etcd-server.key with 2048bit:
```
openssl genrsa -out etcd-server.key 2048
```
2. Generate the certificate signing request based on openssl.cnf:
```
openssl req -new -key etcd-server.key -subj "/CN=etcd-server" -out etcd-server.csr -config openssl.cnf
```

3. Generate the server certificate using the ca-etcd.key, ca-etcd.crt and etcd-server.csr:

```
openssl x509 -req -in etcd-server.csr -CA ca-etcd.crt -CAkey ca-etcd.key -CAcreateserial  -out etcd-server.crt -extensions v3_req -extfile openssl.cnf -days 1000
```
###  ETCD client Certificate

Since the only component that can request the etcd-server is kube-apiserver, we need to generate a certificate signed by the same CA of etcd server.

1. Generate a etcd-client.key with 2048bit:

```
openssl genrsa -out etcd-client.key 2048
```
2. Generate the certificate signing request based on openssl.cnf:

```
openssl req -new -key etcd-client.key -subj "/CN=etcd-client" -out etcd-client.csr -config openssl.cnf
```
3. Generate the server certificate using the ca-etcd.key, ca-etcd.crt and etcd-client.csr:
```
openssl x509 -req -in etcd-client.csr -CA ca-etcd.crt -CAkey ca-client.key -CAcreateserial  -out etcd-client.crt -extensions v3_req -extfile openssl.cnf -days 1000
```


###  The Admin Certificate
Generate a private key and a certificate for the `cluster admin`:

1. Generate a admin.key with 2048bit:
```
$openssl genrsa -out admin.key 2048
```
2. Generate the certificate signing request based on openssl.cnf:

```
$openssl req -new -key admin.key -subj "/CN=kubernetes-admin" -out admin.csr -config openssl.cnf
```

3. Generate the server certificate using the ca.key, ca.crt and admin.csr:
```
$openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out admin.crt -extensions v3_req -extfile openssl.cnf -days 1000
```
###  Kube-controller-manager Certificate
Generate a private key and a certificate for `kube-controller-manager`:

1. Generate a kube-controller-manager.key with 2048bit:

```
openssl genrsa -out kube-controller-manager.key 2048
```
2. Generate the certificate signing request based on openssl.cnf:

```
openssl req -new -key kube-controller-manager.key -subj "/CN=kube-controller-manager" -out kube-controller-manager.csr -config openssl.cnf
```
3. Generate the server certificate using the ca.key, ca.crt and kube-controller-manager.csr:

```
openssl x509 -req -in kube-controller-manager.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out kube-controller-manager.crt -extensions v3_req -extfile openssl.cnf -days 1000
```

###  Kube-scheduler Certificate

Generate a private key and a certificate for `kube-scheduler`:

1. Generate a kube-scheduler.key with 2048bit:

```
openssl genrsa -out kube-scheduler.key 2048
```
2. Generate the certificate signing request based on openssl.cnf:

```
openssl req -new -key kube-scheduler.key -subj "/CN=kube-scheduler" -out kube-scheduler.csr -config openssl.cnf
```
3. Generate the server certificate using the ca.key, ca.crt and kube-scheduler.csr :

```
openssl x509 -req -in kube-scheduler.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out kube-scheduler.crt -extensions v3_req -extfile openssl.cnf -days 1000
```

###  Kube-proxy Certificate

Generate a private key and a certificate for `kube-proxy`:

1. Generate a kube-proxy.key with 2048bit:

```
openssl genrsa -out kube-proxy.key 2048
```
2. Generate the certificate signing request based on openssl.cnf:

```
openssl req -new -key kube-proxy.key -subj "/CN=kube-proxy" -out kube-proxy.csr -config openssl.cnf
```
3. Generate the server certificate using the ca.key, ca.crt and kube-proxy.csr:

```
openssl x509 -req -in kube-proxy.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out kube-proxy.crt -extensions v3_req -extfile openssl.cnf -days 1000
```

###  Kubelet Certificate

Generate a private key and a certificate for `kubelet`:

1. Generate a kubelet.key with 2048bit:

```
openssl genrsa -out kubelet.key 2048
```
2. Generate the certificate signing request based on openssl.cnf:

```
openssl req -new -key kubelet.key -subj "/CN=kubelet" -out kubelet.csr -config openssl.cnf
```
3. Generate the server certificate using the ca.key, ca.crt and kubelet.csr:

```
openssl x509 -req -in kubelet.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out kubelet.crt -extensions v3_req -extfile openssl.cnf -days 1000
```

###  Kube-apiserver Certificate

Generate a private key and a certificate for `kube-apiserver`:

1. Generate a kube-apiserver.key with 2048bit:

```
openssl genrsa -out kube-apiserver.key 2048
```
2. Generate the certificate signing request based on openssl.cnf:

```
openssl req -new -key kube-apiserver.key -subj "/CN=kube-apiserver" -out kube-apiserver.csr -config openssl.cnf
```
3. Generate the server certificate using the ca.key, ca.crt and kube-apiserver.csr:

```
openssl x509 -req -in kube-apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out kube-apiserver.crt -extensions v3_req -extfile openssl.cnf -days 1000
```

Next step will be to add the same parameters when starting kubernetes components binary .
