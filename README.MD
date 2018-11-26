

# 0. Prerequsite
## 0-1. Clone this repo
```
$ git clone https://github.com/komushi/aws-eks-handson
```

## 0-2. Clone the Service Repos
```
$ mkdir environment
$ cd ./environment
$ git clone https://github.com/brentley/ecsdemo-nodejs.git
$ git clone https://github.com/brentley/ecsdemo-crystal.git
$ git clone https://github.com/brentley/ecsdemo-frontend.git
```

## 0-3. Install/Upgrade kubectl
```
$ brew install kubernetes-cli
```

OR
```
$ brew upgrade kubernetes-cli
```

## 0-4. Install ekscli
```
$ brew install weaveworks/tap/eksctl
```

## 0-5. Ensure the ELB Service Role exists
```
aws iam get-role --role-name "AWSServiceRoleForElasticLoadBalancing" || aws iam create-service-linked-role --aws-service-name "elasticloadbalancing.amazonaws.com"
```

# 1. Create an EKS cluster
## 1-1. Create a cluster by eksctl
**cluster-id:eks-workshop**
```
$ eksctl create cluster --name=eks-workshop --nodes=3 --node-ami=auto --region=us-east-2

$ eksctl create cluster --name=eks-workshop --nodes=3 --node-ami=auto --region=us-east-1 --zones=us-east-1a,us-east-1b,us-east-1c

```

**Check ~/.kube/config after cluster creation**
```
$ cat  ~/.kube/config
```

**If no value retrieve it**
```
$ eksctl utils write-kubeconfig --name=eks-workshop --region=us-east-2
```

## 1-2. Test the cluster
```
$ kubectl get nodes
NAME                              STATUS   ROLES    AGE   VERSION
ip-192-168-127-144.ec2.internal   Ready    <none>   18m   v1.10.3
ip-192-168-128-244.ec2.internal   Ready    <none>   18m   v1.10.3
ip-192-168-212-229.ec2.internal   Ready    <none>   18m   v1.10.3
```

## 1-3. Check all the info from the cluster
**namespace: kube-system**
```
$ kubectl get all -n kube-system
```

**namespace: default**
```
$ kubectl get all -n default
```

# 2. Dashboard
## 2-1. Deploy the Kubernetes dashboard
```
$ kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml

secret/kubernetes-dashboard-certs created
serviceaccount/kubernetes-dashboard created
role.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
deployment.apps/kubernetes-dashboard created
service/kubernetes-dashboard created
```

## 2-2. Check all the info from the cluster
**namespace: kube-system**
```
$ kubectl get all -n kube-system
```

**namespace: default**
```
$ kubectl get all -n default
```

## 2-3. Start a local proxy to connect to EKS
```
$ kubectl proxy --port=8888 --address='0.0.0.0' --disable-filter=true &
```

## 2-4. Check the API
[API URL](http://localhost:8080/)
```
{
  "paths": [
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    "/apis/admissionregistration.k8s.io",
    "/apis/admissionregistration.k8s.io/v1beta1",
    "/apis/apiextensions.k8s.io",
    "/apis/apiextensions.k8s.io/v1beta1",
    "/apis/apiregistration.k8s.io",
    "/apis/apiregistration.k8s.io/v1",
    "/apis/apiregistration.k8s.io/v1beta1",
    "/apis/apps",
    "/apis/apps/v1",
    "/apis/apps/v1beta1",
    "/apis/apps/v1beta2",
    "/apis/authentication.k8s.io",
    "/apis/authentication.k8s.io/v1",
    "/apis/authentication.k8s.io/v1beta1",
    "/apis/authorization.k8s.io",
    "/apis/authorization.k8s.io/v1",
    "/apis/authorization.k8s.io/v1beta1",
    "/apis/autoscaling",
    "/apis/autoscaling/v1",
    "/apis/autoscaling/v2beta1",
    "/apis/batch",
    "/apis/batch/v1",
    "/apis/batch/v1beta1",
    "/apis/certificates.k8s.io",
    "/apis/certificates.k8s.io/v1beta1",
    "/apis/events.k8s.io",
    "/apis/events.k8s.io/v1beta1",
    "/apis/extensions",
    "/apis/extensions/v1beta1",
    "/apis/networking.k8s.io",
    "/apis/networking.k8s.io/v1",
    "/apis/policy",
    "/apis/policy/v1beta1",
    "/apis/rbac.authorization.k8s.io",
    "/apis/rbac.authorization.k8s.io/v1",
    "/apis/rbac.authorization.k8s.io/v1beta1",
    "/apis/storage.k8s.io",
    "/apis/storage.k8s.io/v1",
    "/apis/storage.k8s.io/v1beta1",
    "/healthz",
    "/healthz/autoregister-completion",
    "/healthz/etcd",
    "/healthz/ping",
    "/healthz/poststarthook/apiservice-openapi-controller",
    "/healthz/poststarthook/apiservice-registration-controller",
    "/healthz/poststarthook/apiservice-status-available-controller",
    "/healthz/poststarthook/bootstrap-controller",
    "/healthz/poststarthook/ca-registration",
    "/healthz/poststarthook/generic-apiserver-start-informers",
    "/healthz/poststarthook/kube-apiserver-autoregistration",
    "/healthz/poststarthook/rbac/bootstrap-roles",
    "/healthz/poststarthook/start-apiextensions-controllers",
    "/healthz/poststarthook/start-apiextensions-informers",
    "/healthz/poststarthook/start-kube-aggregator-informers",
    "/healthz/poststarthook/start-kube-apiserver-informers",
    "/metrics",
    "/openapi/v2",
    "/swagger-2.0.0.json",
    "/swagger-2.0.0.pb-v1",
    "/swagger-2.0.0.pb-v1.gz",
    "/swagger.json",
    "/swaggerapi",
    "/version"
  ]
}
```

## 2-5. Generate the token for the dashboard
**Use 'eks-workshop' as the cluster-id which was the cluster name created at 1-1.**
```
$ heptio-authenticator-aws token -i eks-workshop
{"kind":"ExecCredential","apiVersion":"client.authentication.k8s.io/v1alpha1","spec":{},"status":{"token":"k8s-aws-v1.aHR0cHM6Ly9zdHMuYW1hem9uYXdzLmNvbS8_QWN0aW9uPUdldENhbGxlcklkZW50aXR5JlZlcnNpb249MjAxMS0wNi0xNSZYLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFKSUczWVRHS1pRTE5NUFBRJTJGMjAxODEwMTglMkZ1cy1lYXN0LTElMkZzdHMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDE4MTAxOFQwMzEyMjdaJlgtQW16LUV4cGlyZXM9NjAmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JTNCeC1rOHMtYXdzLWlkJlgtQW16LVNpZ25hdHVyZT1jZDJhNjJhMzZiNjhjMGU4YWFmNmMzZTFhMzAzMWUzN2ZlMDA2ZTNkN2M1MTA1N2Y3MjJlZDQwYzdjYTIzY2Uy"}}
```

## 2-6. Access the dashboard
[Dashboard URL](http://localhost:8888/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/)

**Use the token created in 2-5 to access the dashboard**

# 3. Deploy the NodeJS Backend API
## 3-1. Check the deployment.yaml and service.yaml
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

## 3-2. Deploy with kubectl
```
$ cd ./environment/ecsdemo-nodejs
$ kubectl apply -f kubernetes/deployment.yaml
$ kubectl apply -f kubernetes/service.yaml
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

## 3-3. Scale the NodeJS Backend API by deployment.yaml
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
$ cd ./environment/ecsdemo-nodejs
$ kubectl apply -f kubernetes/deployment.yaml
```

**Check the deployment**
```
$ kubectl get deployment
NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
ecsdemo-nodejs   3         3         3            3           1h
```

# 4. Deploy the Crystal Backend API
## 4-1. Deploy with kubectl
```
$ cd ./environment/ecsdemo-crystal
$ kubectl apply -f kubernetes/deployment.yaml
$ kubectl apply -f kubernetes/service.yaml
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

# 5. Deploy Frontend Service
## 5-1. Deploy with kubectl
```
$ cd ./environment/ecsdemo-frontend
$ kubectl apply -f kubernetes/deployment.yaml
$ kubectl apply -f kubernetes/service.yaml
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

# 6. Use the application and check scalability and high-availability
## 6-1. Open the URL from 5-1
[elb-aname](http://elb-aname)

## 6-2. Scale the Crystal Backend API
```
$ kubectl scale deployment ecsdemo-crystal --replicas=3
```

## 6-3. Scale the Frontend
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

# 7. Apply NGINX CLB Ingress Controller for the Frontend
**TODO move CLB to another MD**
## 7-1. Install NGINX Ingress Controller for AWS

## 7-2. Compose ingress.yaml of the Frontend

## 7-3. Edit service.yaml of the Frontend

## 7-4. Apply and test with Ingress

# 8. Cleanup
## 8-1. Undeploy the applications
```
$ cd ./environment/ecsdemo-frontend
$ kubectl delete -f kubernetes/service.yaml
$ kubectl delete -f kubernetes/deployment.yaml

$ cd ./environment/ecsdemo-crystal
$ kubectl delete -f kubernetes/service.yaml
$ kubectl delete -f kubernetes/deployment.yaml

$ cd ./environment/ecsdemo-nodejs
$ kubectl delete -f kubernetes/service.yaml
$ kubectl delete -f kubernetes/deployment.yaml

$ kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

**Check the deployment**
```
$ kubectl get deployment
```

**Check the service**
```
$ kubectl get service
```

## 8-2. Delete the EKSCTL Cluster
```
$ eksctl delete cluster --name=eks-workshop --region=us-east-2
```

# Credit: [Amazon EKS Workshop](https://eksworkshop.com/)
# Credit: [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/#aws)

https://medium.com/@sevcsik/authentication-using-https-client-certificates-3c9d270e8326

https://qiita.com/sheepland/items/37ece5800eb6972ffd91