# WIP: Don't use it yet!


https://github.com/arun-gupta/istio-kubernetes-aws#apply-default-destination-rules

https://medium.com/@mahesh.rayas/istio-deploy-sample-bookinfo-to-aws-eks-543b59474783



```
brew install kubernetes-helm
```

# 1. Create an EKS cluster
## 1-1. Create a cluster by eksctl
**cluster-id:istio-eks**
```
$ eksctl create cluster --name=eks-istio --nodes=4 --node-ami=auto --region=us-east-1 --zones=us-east-1a,us-east-1b,us-east-1c,us-east-1d
```

**Check ~/.kube/config after cluster creation**
```
$ cat  ~/.kube/config
```

## 1-2. Test the cluster
```
$ kubectl get nodes
NAME                              STATUS   ROLES    AGE   VERSION
ip-192-168-138-81.ec2.internal    Ready    <none>   1m    v1.10.3
ip-192-168-160-145.ec2.internal   Ready    <none>   1m    v1.10.3
ip-192-168-218-112.ec2.internal   Ready    <none>   1m    v1.10.3
ip-192-168-93-34.ec2.internal     Ready    <none>   1m    v1.10.3
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

# 2. Install and Configure Istio
## 2-1. Download latest Istio
```
$ mkdir istio
$ cd ./istio
$ curl -L https://git.io/getLatestIstio | sh -
$ cd istio-1.0.2
```

## 2-2. Install helm ServiceAccount
```
$ kubectl create -f install/kubernetes/helm/helm-service-account.yaml
```

```
$ helm init --service-account tiller
```

## 2-3. Install Istio
```
helm install --wait --name istio --namespace istio-system install/kubernetes/helm/istio --set tracing.enabled=true --set grafana.enabled=true
```

## 2-4. Verify services
```
$ kubectl get pods -n istio-system
```

## 2-5. Verify services
```
$ kubectl label namespace default istio-injection=enabled
```

# 3. Istio BookInfo Application
https://istio.io/docs/examples/bookinfo/
## 3-1. Deploy BookInfo
### 3-1-1. Deploy the application with automatic sidecar injection for each pod
```
$ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml 
```

### 3-1-2. Verify services
```
$ kubectl get svc
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
details       ClusterIP   10.100.43.195    <none>        9080/TCP   1h
kubernetes    ClusterIP   10.100.0.1       <none>        443/TCP    1h
productpage   ClusterIP   10.100.100.186   <none>        9080/TCP   1h
ratings       ClusterIP   10.100.113.190   <none>        9080/TCP   1h
reviews       ClusterIP   10.100.164.187   <none>        9080/TCP   1h
```

### 3-1-3. Verify pods
```
$ kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
details-v1-6865b9b99d-dxrqk    2/2     Running   0          1h
productpage-v1-f8c8fb8-7ds9f   2/2     Running   0          1h
ratings-v1-77f657f55d-9dkg6    2/2     Running   0          1h
reviews-v1-6b7f6db5c5-tk8cj    2/2     Running   0          1h
reviews-v2-7ff5966b99-dc2vn    2/2     Running   0          1h
reviews-v3-5df889bcff-gdqkt    2/2     Running   0          1h
```

## 3-2. Define Ingress
```
$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
$ kubectl get gateway
```

## 3-3. Access the BookInfo Application
**Get the LoadBalancer External IP and Port**
```
$ kubectl get service -n istio-system
```
http://xxxx.us-east-1.elb.amazonaws.com/productpage
https://xxxx.us-east-1.elb.amazonaws.com/productpage


---------
### clean up
```
kubectl delete -f install/kubernetes/helm/istio/templates/crds.yaml -n istio-system
```

```
kubectl delete -f install/kubernetes/helm/helm-service-account.yaml
```