# Kustomize workshop

In this workshop you will learn how to use Kustomize. 

## Usecase
Company want you to release a application for production with text `Welcome to production` with a total of 3 replicas and the beta release with the text `Welcome on our beta website`. Since the webserver will be nginx for both your task is to use kustomize. 

## Create a base for kustomize
The basis of kustomize is the part that should be the same for all deployments. This is where you'll begin. After you've created the base, you'll begin adding overlays for various purposes.
The kustomize base is made up of the yaml code that you want to deploy. as well as a kustomization.yaml file were you specify which resources you want to include in this file.

example: 
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
metadata:
  name: base
commonLabels:
  app: nginx-deployment
resources:
  - nginxDeploy.yaml
  - service.yaml
  - index-html-configmap.yaml
```

## Creating a overlay for kustomize to overwrite the content of the website 
Start by creating a new folder 'production' for example. In this folder you will add a kustomizaition.yaml where in you will specify which resources you wanna add or overwrite. 

example:
```
---
namePrefix: production-
commonLabels:
  variant: production
commonAnnotations:
  note: Hello, I am production!
resources:
  - ../../base
patchesStrategicMerge:
  - index-html-configmap.yaml
  - nginxDeploy.yaml
```

Create two overlays: one for production and one for testing. We intend to deploy more replicas in production than in the beta version. And, of course, have a different webpage page.

When you have create your overlay you can apply it by using the following command:
```
kubectl apply -k overlay/production 
```

To see you webpage use the kubectl portforward:
```
kubectl port-forward service/production-nginx-service 8033:80
curl http://localhost:8033/index.html
```