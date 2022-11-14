# service mesh with istio 
This shows you how to setup service mesh with istio. and do traffic splitsing. 
## requirements 
* helm 
* kubernetes cluster where you can reach the ingress (minikube tunnel)


# Installing istio using helm

To have a service mesh setup for kubernetes you need to first install istio. We are gonna use helm for this installation. 

Start by adding the istio repository to our helm application:
`
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
`


Starting to install istio in our cluster. For istio to work we will need 3 components of isio. 
* istio-init 
* istio-base
* istio/istiod
We will install this in the istio-system namespace. 
using the following commands: 
```
kubectl create ns istio-system
helm install istio-init --namespace istio-system istio.io/istio-ini
helm install istio-base istio/base -n istio-system
helm install istiod istio/istiod -n istio-system --wait
```

Now that istio components are install we need to create a ingress to reach our services. But this time you don't create an ingress from kubernetes. but you will create a istio gateway. to have this separated we will install in it's own namespace.

```
kubectl create namespace istio-ingress
kubectl label namespace istio-ingress istio-injection=enabled
helm install istio-ingress istio/gateway -n istio-ingress --wait
helm install istio-ingressgateway istio/gateway -n istio-ingress
kubectl label namespace default istio-injection=enabled --overwrite
```
In the commands you also see we enable istio-injection. This is to take advantage of all the features of istio. This will make sure that all the pods in that namespace will be running istio sidecar proxy. 

Now we have our environment ready to install our nginx production and beta.

```
kubectl apply -k ../kustomize/overlays/beta
kubectl apply -k ../kustomize/overlays/production
```


But now we want to start routing traffic to our production application. to do this we need to setup gateway and VirtualService.  This are 2 yamls that you will need to apply to the cluster. 
The gateway setup is using app selector to point to the gateway we create as our ingress.  here by we allow http traffic on port 80 for all hosts. 
Gateway.yml:
```
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: nginx-gateway
spec:
  selector:
    app: istio-ingressgateway
  servers:
    - port:
        number: 80
        name: http
        protocol: HTTP
      hosts:
        - "*"
```

virtualservice.yml
```
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: nginx-service-ingress
spec:
  hosts:
    - "*"
  gateways:
    - nginx-gateway
  http:
    - route:
        - destination:
            host: production-nginx-service
```

Now apply both yaml files using kubectl. 

When applied viste localhost website. 
```
$ curl localhost
output: 
<html>
<h1>Welcome to production</h1>
</br>
<h1>This is a working nginx website on kubernetes! </h1>
</html>
```


## traffic shifting
Now we have it production working. But we need to test in production if beta version will also work so we gonna send 10 %  of our traffic to beta version. 
This is done by adding a weight value in virtualservice route. 

virtualservice.yml
```
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: nginx-service-ingress
spec:
  hosts:
    - "*"
  gateways:
    - nginx-gateway
  http:
    - route:
        - destination:
            host: production-nginx-service
          weight: 90
        - destination:
            host: beta-nginx-service
          weight: 10
```

Now when you refresh the webpage a couple of time it should also show the beta version.