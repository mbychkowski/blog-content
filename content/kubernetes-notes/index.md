---
title: Public notes on Kubernetes
date: "2020-03-22T00:00:00.000Z"
description: "Create a blog and deploy with S3"
---

## Installs
# Setting up a cluster
* Docker
* Kubeadm
* Kubelet
* Kubectl
* Control Plane (only for master node)

* __Docker__ - Container runtime. Runs containers on each node. [Docs](https://docs.docker.com/install/linux/docker-ce/ubuntu/). To use docker with `kubeadm` need to install specific version of docker. `kubeadm` goes through vetting process that does not always allow latest and greatest version of docker to be used `sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu`. To prevent an auto update on docker run: `sudo apt-mark hold docker-ce`
* __Kubernetes (`kubeadm`, `kubelet`, `kubectl`)__ - [Docs](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl). `Kubeadm` automates a large portion of the effort for standing up a kubernetes cluster. `Kubelet` essential piece. Needs to be installed. An agent that serves as a middleman between containers and kubernetes. `Kubectl` is a command line tool on how we can interact with a cluster. To install these 3 items it is best to use the current versions `sudo apt-get install -y kubelet=1.12.7-00 kubeadm=1.12.7-00 kubectl=1.12.7-00`, and hold version `sudo apt-mark hold kubelet kubeadm kubectl`

## Bootstrapping a cluster
1. Initialize a cluster with cidr commnad on __master__ node: `sudo kubeadm init --pod-network-cidr=10.244.0.0/16`.
2. Part of the output from step (1.) will allow us to setup `kubectl` locally. We can copy the following and run on our master node: 
```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
After this step our kubernetes master node should be up and running.
3. Joining kubernetes worker nodes to the cluster. To do this the command is also part of the output from step (1.). It should resemble something like the following: `sudo kubeadm join 172.31.41.75:6443 --token knqjmo.yfvkkmm88pb4hdhn --discovery-token-ca-cert-hash sha256:5e358e774b3a6a59c134fe74c99619baee0c4662d4d74de8335afc37d5b81e16`. Run this command for both worker nodes to connect to the cluster.

## Configure networking with Flannel
1. Run the following on master and each worker node:
`echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf`
`sudo sysctl -p`

2. Run only on master node: `kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml`

Afterwards the status state should be "Ready".

# Terminolog
__Pod__ smallest entity in Kubernetes. Typically you will have 1 container per pod, but can have more than 1. In order for a container to be connected to cluster it must be in a pod. Kubernetes schedules Pods to run on Nodes.
__Nodes__ are the servers that are actually running the containers. The control server (master node?) hosts the Kubernetes API. The worker nodes are what is actually running the applications.