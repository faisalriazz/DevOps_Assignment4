# Assignment 04 - Minikube
* Install and configure Minikube.
  
  Minikube is installed on windows using the below mentioned link
  <https://minikube.sigs.k8s.io/docs/start/> selecting .exe donwload. 

  ```minkube start```
  
``` $ minikube start
* minikube v1.31.2 on Microsoft Windows 10 Pro 10.0.19045.3324 Build 19045.3324
* Using the hyperv driver based on existing profile
* Starting control plane node minikube in cluster minikube
* Restarting existing hyperv VM for "minikube" ...
* Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...
* Configuring bridge CNI (Container Networking Interface) ...
* Verifying Kubernetes components...
  - Using image docker.io/kubernetesui/dashboard:v2.7.0
  - Using image docker.io/kubernetesui/metrics-scraper:v1.0.8
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Some dashboard features require the metrics-server addon. To enable all features please run:

  minikube addons enable metrics-server


* Enabled addons: storage-provisioner, default-storageclass, dashboard
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

* Create deployments using custom Docker images.

  used custom docker image for sample node app. deployment and service yaml file is shown below:


```$ cat node-app-deployment.yml```
```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: sample-node-app
  labels:
  namespace:  default

spec:
  replicas: 1
  selector:
    matchLabels:
      app: sample-node-app

  template:
    metadata:
      labels:
        app: sample-node-app

    spec:
      containers:
      - name: sample-node-app
        image: faisalriazz/my-app-node:1.0
        ports:
        - containerPort: 3000

#service to expose external interface.
---
apiVersion: v1
kind: Service

metadata:
  name: node-app-service

spec:
  selector:
    app: sample-node-app

  type: LoadBalancer

  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
```
* Set up various types of services (NodePort, ClusterIP, LoadBalancer) to manage access to the deployed pods.
creating the deployment using the above yaml file where service 
type is LoadBalancer.

```$ kubectl apply -f node-app-deployment.yml```

```
deployment.apps/sample-node-app created
service/node-app-service created
```
check the status of deployment and service
```
faisa@FaisalBhatti MINGW64 /d/DiceCamp/DevOps_Assignment4 (main)
$ kubectl get deployment
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
sample-node-app   1/1     1            1           2m47s

faisa@FaisalBhatti MINGW64 /d/DiceCamp/DevOps_Assignment4 (main)
$ kubectl get service
NAME               TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes         ClusterIP      10.96.0.1        <none>        443/TCP        30h
node-app-service   LoadBalancer   10.100.242.127   <pending>     80:31628/TCP   2m55s

```
Running the service for accessing the app outside the cluster.
```
faisa@FaisalBhatti MINGW64 /d/DiceCamp/DevOps_Assignment4 (main)
$ minikube service node-app-service
|-----------|------------------|-------------|----------------------------|
| NAMESPACE |       NAME       | TARGET PORT |            URL             |
|-----------|------------------|-------------|----------------------------|
| default   | node-app-service |          80 | http://172.22.225.70:31628 |
|-----------|------------------|-------------|----------------------------|
🎉  Opening service default/node-app-service in default browser...

```
Changing the service type to Node port . by setting a selector. The service exposes port 80 and forwards the traffic to the pods’ port 3000.

```yaml
apiVersion: v1
kind: Service

metadata:
  name: node-app-service

spec:
  selector:
    app: sample-node-app

  type: NodePort

  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
```

```
faisa@FaisalBhatti MINGW64 /d/DiceCamp/DevOps_Assignment4 (main)
$ kubectl apply -f node-app-deployment-nodeport.yml
deployment.apps/sample-node-app created
service/node-app-service created

faisa@FaisalBhatti MINGW64 /d/DiceCamp/DevOps_Assignment4 (main)
$ kubectl get service
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes         ClusterIP   10.96.0.1        <none>        443/TCP        30h
node-app-service   NodePort    10.111.249.172   <none>        80:30956/TCP   10s

faisa@FaisalBhatti MINGW64 /d/DiceCamp/DevOps_Assignment4 (main)
$ minikube service node-app-service
|-----------|------------------|-------------|----------------------------|
| NAMESPACE |       NAME       | TARGET PORT |            URL             |
|-----------|------------------|-------------|----------------------------|
| default   | node-app-service |          80 | http://172.22.225.70:30956 |
|-----------|------------------|-------------|----------------------------|
🎉  Opening service default/node-app-service in default browser...

```



   

  