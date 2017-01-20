# Continuous Delivery in Kubernetesland

This repository contains all code related to the training Continuous Delivery in Dockerland by [Nicolas Byl](http://lanyrd.com/profile/nicolas.byl/). In this session we will provide a step-by-step demo how to build a continuous delivery pipeline running on [kubernetes](http://kubernetes.io).

## Lab 0: Prerequisites

To have this running on your machine, you will need the following tools:

* [minikube](https://github.com/kubernetes/minikube/releases)
* [kubectl](https://kubernetes.io/docs/getting-started-guides/kubectl/)
* [helm](https://github.com/kubernetes/helm/releases)

You will also need a virtualization that is [supported by minikube](https://github.com/kubernetes/minikube#installation). This tutorial has been tested on [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

## Lab 1: Setup

### Kubernetes
First of all initialize your kubernetes cluster with minikube:

    minikube addons enable dashboard
    minikube addons enable ingress
    minikube start --memory 4096

Maybe, you will have to select your VM driver using `--vm-driver`. To see your available options, please refer to `minikube start --help`.    

After the cluster has been started, verify your installation with:

    kubectl cluster-info

The ip of your minikube VM can be found using:

    minikube ip

**Update**: If you want to download the docker images on high-speed connection now, you can issue the following command:

    minikube ssh 'gcr.io/kubernetes-helm/tiller:v2.8.1 nbyl/jenkins-slave-docker-image-build:3.10-2 java:8 postgres:9.5.4 jenkins/jenkins:lts jenkins/jnlp-slave:3.10-1; do docker pull $i; done'

### Helm    

Initialize helm using it's init command:

    helm init

When the server component tiller is ready, you can view a list of your (not yet existing) applications with:

    helm list

### Jenkins

Now that helm and kubernetes are ready, we can install Jenkins using the a helm command:

    helm install stable/jenkins --set Agent.Memory=1024Mi,Persistence.StorageClass=standard --name=cd

Now you can search for the port of the jenkins using:

    kubectl describe svc cd-jenkins |grep NodePort|grep http

Pass the resulting port into your browser using the URL http://<minikube IP>:<Port>. The user to login is `admin` and the password can be found the command given by the output of `helm install ...` earlier.
