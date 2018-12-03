# WIP: Don't use it yet!

## 1-2. Option 2 install with Helm
### 1-2-1. Install and initialize Helm
[Install and initialize Helm](HELM.MD)

### 1-2-2. Install chart alb-ingress-controller-helm
```
$ helm registry install quay.io/coreos/alb-ingress-controller-helm --namespace kube-system --name=alb-ingress -f alb-ingress/values.yaml
...
...
```

**Check the release by Helm**
```
$ helm list
NAME        REVISION  UPDATED                   STATUS    CHART                             APP VERSION NAMESPACE  
alb-ingress 1         Wed Nov 21 15:37:12 2018  DEPLOYED  alb-ingress-controller-helm-0.1.3 v1.0.0      kube-system
```

**Check the ServiceAccount, the ClusterRoleBinding and the ClusterRole**
```
$ kubectl get clusterrole -n=kube-system
$ kubectl get clusterrolebinding -n=kube-system
$ kubectl get serviceaccount -n=kube-system
```

**Check the deployment was successful and the controller started**
```
$ kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o alb-ingress[a-zA-Z0-9-]+)
-------------------------------------------------------------------------------
AWS ALB Ingress controller
  Release:    v1.0.0
  Build:      git-6ee1276
  Repository: https://github.com/kubernetes-sigs/aws-alb-ingress-controller
-------------------------------------------------------------------------------
```

**Check the status of the alb-ingress-controller pod**
```
$ kubectl --namespace=kube-system get pods -l "app=alb-ingress-controller,component=controller,release=alb-ingress"
NAME                                                             READY   STATUS    RESTARTS   AGE
alb-ingress-alb-ingress-controller-controller-64d4546948-922bg   1/1     Running   0          11m
```


## 3-2. Delete chart alb-ingress-controller-helm
```
$ helm delete --purge alb-ingress
```


----------------------
# Delete chart alb-ingress-controller-helm
```
$ helm delete --purge alb-ingress
```