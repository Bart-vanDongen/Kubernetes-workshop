# Kubernetes-workshop

This repo is used for a workshop about kubernetes.


# Requirements 

* Docker 
* [Kind](https://kind.sigs.k8s.io/) / [minikube](https://minikube.sigs.k8s.io/docs/start/) (or other kubernetes clusters)
* helm 
* kubectl 

## Preparation

Install kind:

https://kind.sigs.k8s.io/docs/user/quick-start/#installation


Install kubectl: 

https://kubernetes.io/docs/tasks/tools/


Install helm: 

https://helm.sh/docs/intro/install/

Install minikube:

https://minikube.sigs.k8s.io/docs/start/
# Workshop 

This workshop contains 3 elements. 
* How to deploy an web application using yaml's
* How to deploy different versions of the web application using kustomize
* Service mesh (using helm)


## Topic links: 
1. [web application](web-application/README.md)
2. [Kustomize](kustomize/README.md)
3. [Service mesh](service-mesh/README.md)
