# Kubernetes Anatomy

[k8s]: https://assets.sumologic.com/application-content/Kubernetes-Logo-300x300.png 

Kubernetes or k8s abstracts the underlying infrastructure by providing us with a simple API to which we can send requests but in this tutorial we are going into the dirty work of kubernetes. If you are looking for a ready-to-use k8s cluster for development or test perpose , I recommend using [Minikube](https://kubernetes.io/fr/docs/tasks/tools/install-minikube/).

By learning k8s anatomy you will be able to bootstrap a kubernetes cluster with end-to-end encryption between components.


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


