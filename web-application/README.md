# Web-application deployment kubernetes
You wanna create your first web application in a kubernetes clusters. 
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
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
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