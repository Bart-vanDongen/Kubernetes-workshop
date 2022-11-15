# Web-application deployment kubernetes
You wanna create your first web application in a kubernetes clusters. 

## Create cluster
Minikube cluster is recommended. Because this is required for service mesh istio.

Create a minikube cluster by using the following command.

```
minikube start
```

kind create cluster using the following command:

```
kind create cluster
```


## Start of by creating a workload for kubernetes
This can be accomplished in a variety of ways. However, we shall begin by making a pod.

Make a yaml file called nginxPod.yaml. We will define our container information in this yaml file.

Content for nginxPod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

Now you have your yaml file ready. You can use kubectl to apply your yaml on to your cluster. 

```
kubectl apply -f nginxPod.yaml
```
To see your pod running use the following command
```
kubectl get pods
```

to connect to the nginx web page you can use the forward command of kubectl
```
kubectl port-forward pods/nginx 8000:80
```

## Going from pod to deployment
Now we'll transition from a pod to a deployment.
This is done so that we may explain the desired condition of a deployment. What we want to define is how many replicas of the nginx pods we want to run. Rather than deploying nginxPod.yaml three times, we will specify in deployment that we want three replicas.

We'll begin by building nginxDeploy. You can use the nginxPod in yaml. Use yaml as your starting point. and begin by altering the type to deployment

The container information is specified in the template section. relocating the spec section to the deployment template section



```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

Now you have your yaml file ready. You can use kubectl to apply your yaml on to your cluster. 

```
kubectl apply -f nginxDeploy.yaml
```
To see your deployment in kubernetes use the following command:
```
kubectl get deployment
```

To see your pod running use the following command
```
kubectl get pods
```


## Create a service for your deployment
In Kubernetes, a service is an abstract mechanism to expose a network service for an application operating on a number of pods.

Create a service.yaml file that uses selector to point to the label that you used in nginxDeploy.yaml


Forward to service port to see if it actually worked:
```
kubectl port-forward service/nginx-service 8033:80
```


## EXTRA: update the content of nginx web server.

This is accomplished by include a config map including an index.html file in kubernetes. After that, mount it as a volume.

Example configmap.yml: 
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: index-page
data:
  index.html: |
    <html>
    <h1>Welcome to the workshop</h1>
    </br>
    <h1>This is a working nginx website on kubernetes! </h1>
    </html
```


example of nginx deployment:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
          volumeMounts:
            - name: nginx-index-file
              mountPath: /usr/share/nginx/html/
      volumes:
        - name: nginx-index-file
          configMap:
            name: index-page
```





## Sources: 
* https://kubernetes.io/docs/concepts/workloads/pods/
* https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
* https://kubernetes.io/docs/concepts/services-networking/service/
* https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/