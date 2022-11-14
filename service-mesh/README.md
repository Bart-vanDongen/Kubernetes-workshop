# service mesh with istio 
This shows you how to setup service mesh with istio. and do traffic splitsing. 
## requirements 
* helm 
* kubernetes cluster where you can reach the ingress (minikube tunnel)


# Installing istio using helm

Istio must be installed before you can set up a service mesh for Kubernetes. For this installation, we will use helm.

Begin by including the istio repository in our helm application:
```
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
```


For istio is being installed in our cluster. It requires three components to be installed in order to function.
* istio-init 
* istio-base
* istio/istiod

This will be installed in the istio-system namespace.
using the following commands: 
```
kubectl create ns istio-system
helm install istio-init --namespace istio-system istio.io/istio-ini
helm install istio-base istio/base -n istio-system
helm install istiod istio/istiod -n istio-system --wait
```

# Installing gateway (ingress)
Now that the Istio components have been installed, we must establish an ingress to reach our services. However, you do not build an ingress from kubernetes this time. However, you will establish an Istio gateway. To keep things distinct, we'll install it in its own namespace.

```
kubectl create namespace istio-ingress
kubectl label namespace istio-ingress istio-injection=enabled
helm install istio-ingress istio/gateway -n istio-ingress --wait
helm install istio-ingressgateway istio/gateway -n istio-ingress
kubectl label namespace default istio-injection=enabled --overwrite
```
We also enable istio-injection in the instructions. This is done to take benefit of all of istio's features. This ensures that all pods in that namespace are running istio sidecar proxy.

# Application setup for service mesh
We now have a ready environment for installing nginx production and beta.
```
kubectl apply -k ../kustomize/overlays/beta
kubectl apply -k ../kustomize/overlays/production
```


We now want to begin directing traffic to our production application. To accomplish this, we must first configure the gateway and the VirtualService. You will need to apply these two yamls to the cluster.

The gateway configuration makes use of app selector to direct traffic to the gateway we construct as our ingress. We allow http traffic on port 80 for all hosts here.
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
In kubernetes, the virtualservice is used to route inbound traffic from the gateway to the appropriate service. We specify which gateways and hosts are affected in the specs.
 
virtualservice.yml:
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
We have production running. However, we need to test in production whether the beta version will also work, thus we will send 10% of our traffic to the beta version.
This is accomplished by including a weight value in the virtualservice route.

virtualservice.yml example:
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