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

To open jenkins in your brower simply use minikube:

    minikube service cd-jenkins

The user to login is `admin` and the password can be found the command given by the output of `helm install ...` earlier.

## Lab 2: Continuous Integration

### Lab 2.1: Create your first build pipeline

Create a new pipeline job for our project:

* Click on "New Item" and enter the name of the project "confy"
* As a type select "Pipeline" and click on "OK"
* Configure the following job options:
  * "Build Triggers" &rarr; "Poll SCM" &rarr; "* * * * *"
  * "Pipeline" 
    * "Definition": "Pipeline from SCM" 
    * "SCM": "Git" 
    * "Repository URL": "https://github.com/nbyl/cd-workshop-demo.git"

Afterwards you can click on "Save" and "Build Now" to start your first pipeline run. Congratulations, your pipeline is setup now.

### Lab 2.2: Cache your dependencies

Everytime a build is now started, gradle will redownload all dependencies mentioned in the build.gradle. To save us some time, we will create a persistent volume to cache these.

First create a persistent volume:

  kubectl apply -f minikube/gradle-cache.yml
  
Now configure Jenkins to use it when spawning a new build pod:

* "Manage Jenkins"
* "Configure System"
* Search for "Kubernetes Pod Template"
  * "Add Volume" &rarr; "Persistent Volume Claim"
    * "Claim Name": gradle-cache
    * "Mount path": /home/jenkins/.gradle
    
Now restart the build. It will most likely fail, because minikubes hostPath provisioner will only allow root access to our filesystem. You can correct the rights using:
     
     minikube ssh 'sudo chmod -R 777 /tmp/hostpath_pv'
     
**Warning**: You should not do this on a production system.     

Now run your build again twice. You should notice a speedup on the second run.

## Lab 3: Publish docker container

First we need to enable the build pod to build docker images. Configure Jenkins to mount the corresponding socket:

* "Manage Jenkins"
* "Configure System"
* Search for "Kubernetes Pod Template"
  * Change "Docker Image" to "nbyl/jenkins-slave-docker-image-build:3.10-2"
  * "Add Volume" &rarr; "Host Path Volume"
    * "Host Path": /run/docker.sock
    * "Mount path": /run/docker.sock
  * "Add Volume" &rarr; "Host Path Volume"
    * "Host Path": /var/run/docker.sock
    * "Mount path": /var/run/docker.sock
 
After saving run the pipeline again. It will call another gradle task that builds a docker container. Publishing is currently disabled as we are on a minikube system and the image will always be present. But why don't you implement this as an exercise?

## Lab 4: Integration Testing

There is nothing to change for this lab. But to check out the helm chart, look at the `helm/confy` directory. The deployment is described here as a set of templates for the kubernetes resources. The application is installed using a deployment of the pod to run. The database for the application is configured using environment variables. The instances of the application are combined using a kubernetes service.

## Lab 5: User Acceptance Testing

There is nothing to change for this lab. If you look at the `helm/confy` chart, you will find an ingress resource to access the application. It will available after deployment at http://<minikube IP>/confy-uat

## Lab 6: Production Deployment

There is nothing to change for this lab. After the confirmation the application will be available at http://<minikube IP>/confy
