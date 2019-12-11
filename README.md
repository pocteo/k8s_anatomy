# Kubernetes Anatomy

![k8s](https://www.functionize.com/wp-content/uploads/2018/04/kubernetes-logo2-768x648.png)


Kubernetes or k8s is considered as the leader when it comes to container orchestration platforms. 
Since 2014, k8s become one of the most famous tool in the open source community.

Kubernetes gives you the ability to manage, deploy and scale your containerized applications with ease.If you want to focus on your containerized applications and to take advantages of k8s features you can use a managed service such as Amazon EKS or Google GKE to provide you a highly available, secure, and managed cluster.

However, to be able to monitor and troubleshoot your applications you should have an idea about k8s anatomy.

This is why in this tutorial we are going into the dirty work of kubernetes by bootstrapping a kubernetes cluster with an end-to-end encryption between components..

> If you are looking for a ready-to-use k8s cluster for local development or test purposes , I recommand using [Minikube](https://kubernetes.io/fr/docs/tasks/tools/install-minikube/).


## Target Audience

The target audience for this tutorial is someone who want to figure out how everything fits together in a kubernetes cluster.

## Labs

* [Introduction](docs/introduction.md)
* [Provisioning the CA and Generating TLS Certificates](docs/generate-tls-certificates.md)
* [Generating Kubeconfig files for authentication](docs/generate-kubeconfig-files.md)
* [Bootstrapping the etcd Cluster](docs/bootstrapping-etcd-cluster.md)
* [Bootstrapping the Kubernetes Control Plane](docs/bootstrapping-control-plane.md)
* [Bootstrapping the Kubernetes Worker Nodes](docs/bootstrapping-worker-nodes.md)
* [Configuring kubectl for Remote Access](docs/configure-kubctl.md)
* [Test Scenario](docs/test.md)


