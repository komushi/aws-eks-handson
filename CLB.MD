# Workload via Classic Load Balancer

# 1. Clone the Service Repos
```
$ mkdir environment
$ cd ./environment
$ git clone --depth=1 --branch=master https://github.com/brentley/ecsdemo-nodejs.git && rm -rf ecsdemo-nodejs/.git
$ git clone --depth=1 --branch=master https://github.com/brentley/ecsdemo-crystal.git && rm -rf ecsdemo-crystal/.git
$ git clone --depth=1 --branch=master https://github.com/brentley/ecsdemo-frontend.git && rm -rf ecsdemo-frontend/.git
$ cd ..
```

# 2. Deploy the NodeJS Backend API
## 2-1. Check the deployment.yaml and service.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecsdemo-nodejs
  labels:
    app: ecsdemo-nodejs
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ecsdemo-nodejs
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: ecsdemo-nodejs
    spec:
      containers:
      - image: brentley/ecsdemo-nodejs:latest
        imagePullPolicy: Always
        name: ecsdemo-nodejs
        ports:
        - containerPort: 3000
          protocol: TCP
```

```
apiVersion: v1
kind: Service
metadata:
  name: ecsdemo-nodejs
spec:
  selector:
    app: ecsdemo-nodejs
  ports:
   -  protocol: TCP
      port: 80
      targetPort: 3000
```

## 2-2. Deploy with kubectl
```
$ kubectl apply -f environment/ecsdemo-nodejs/kubernetes/deployment.yaml
$ kubectl apply -f environment/ecsdemo-nodejs/kubernetes/service.yaml
```

**Check the deployment**
```
$ kubectl get deployment
NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
ecsdemo-nodejs   1         1         1            1           1h
```

**Check the service**
```
$ kubectl get service
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
ecsdemo-nodejs    ClusterIP   10.100.236.34    <none>        80/TCP    1h
```

## 2-3. Scale the NodeJS Backend API by deployment.yaml
**Edit the deployment.yaml**
**Set the nodejs deployment replica to 3**
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecsdemo-nodejs
  labels:
    app: ecsdemo-nodejs
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ecsdemo-nodejs
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: ecsdemo-nodejs
    spec:
      containers:
      - image: brentley/ecsdemo-nodejs:latest
        imagePullPolicy: Always
        name: ecsdemo-nodejs
        ports:
        - containerPort: 3000
          protocol: TCP
```

```
$ kubectl apply -f environment/ecsdemo-nodejs/kubernetes/deployment.yaml
```

**Check the deployment**
```
$ kubectl get deployment
NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
ecsdemo-nodejs   3         3         3            3           1h
```

# 3. Deploy the Crystal Backend API
## 3-1. Deploy with kubectl
```
$ kubectl apply -f environment/ecsdemo-crystal/kubernetes/deployment.yaml
$ kubectl apply -f environment/ecsdemo-crystal/kubernetes/service.yaml
```

**Check the deployment**
```
$ kubectl get deployment
NAME              DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
ecsdemo-crystal   1         1         1            1           1h
ecsdemo-nodejs    3         3         3            3           1h
```

**Check the service**
```
$ kubectl get service
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
ecsdemo-crystal   ClusterIP   10.100.185.190   <none>        80/TCP    1h
ecsdemo-nodejs    ClusterIP   10.100.236.34    <none>        80/TCP    1h
```

# 4. Deploy Frontend Service
## 4-1. Deploy with kubectl
```
$ kubectl apply -f environment/ecsdemo-frontend/kubernetes/deployment.yaml
$ kubectl apply -f environment/ecsdemo-frontend/kubernetes/service.yaml
```

**Check the deployment**
```
$ kubectl get deployment
NAME              DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
ecsdemo-crystal    1         1         1            1           1h
ecsdemo-frontend   1         1         1            1           1h
ecsdemo-nodejs     3         3         3            3           1h
```

**Check the service**
```
$ kubectl get service
NAME               TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
ecsdemo-crystal    ClusterIP      10.100.185.190   <none>        80/TCP         1h
ecsdemo-frontend   LoadBalancer   10.100.137.73    <elb-aname>   80:31935/TCP   1h
ecsdemo-nodejs     ClusterIP      10.100.236.34    <none>        80/TCP         1h
```

# 5. Use the application and check scalability and high-availability
## 5-1. Check the demo app on browser
[URL_FROM_4-1](http://)

## 5-2. Scale the Crystal Backend API
```
$ kubectl scale deployment ecsdemo-crystal --replicas=3
```

## 5-3. Scale the Frontend
```
$ kubectl scale deployment ecsdemo-frontend --replicas=3
```

**Check the deployment**
```
$ kubectl get deployment
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
ecsdemo-crystal    3         3         3            3           1h
ecsdemo-frontend   3         3         3            3           1h
ecsdemo-nodejs     3         3         3            3           1h
```

# 6. Clean-up
## 6-1. Undeploy the applications
```
$ kubectl delete -f environment/ecsdemo-frontend/kubernetes/service.yaml
$ kubectl delete -f environment/ecsdemo-frontend/kubernetes/deployment.yaml

$ kubectl delete -f environment/ecsdemo-crystal/kubernetes/service.yaml
$ kubectl delete -f environment/ecsdemo-crystal/kubernetes/deployment.yaml

$ kubectl delete -f environment/ecsdemo-nodejs/kubernetes/service.yaml
$ kubectl delete -f environment/ecsdemo-nodejs/kubernetes/deployment.yaml
```

**Check the deployment**
```
$ kubectl get deployment,service
```
