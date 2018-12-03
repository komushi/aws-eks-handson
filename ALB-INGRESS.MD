# 1. Apply NGINX ALB Ingress Controller for the Frontend
## 1-1. Option 1 install with Helm
### 1-1-1. Install and initialize Helm
[Install and initialize Helm](HELM.MD)

### 1-1-2. Install chart alb-ingress-controller-helm
```
$ helm registry install quay.io/coreos/alb-ingress-controller-helm --namespace kube-system --name=alb-ingress -f alb-ingress/values.yaml
NAME:   alb-ingress
LAST DEPLOYED: Wed Nov 21 15:00:04 2018
NAMESPACE: kube-system
STATUS: DEPLOYED

RESOURCES:
==> v1beta1/Deployment
NAME                                           AGE
alb-ingress-alb-ingress-controller-controller  1s

==> v1/Pod(related)

NAME                                                            READY  STATUS             RESTARTS  AGE
alb-ingress-alb-ingress-controller-controller-64d4546948-zr4br  0/1    ContainerCreating  0         1s

==> v1/ServiceAccount

NAME                                AGE
alb-ingress-alb-ingress-controller  1s

==> v1beta1/ClusterRole
alb-ingress-alb-ingress-controller  1s

==> v1beta1/ClusterRoleBinding
alb-ingress-alb-ingress-controller  1s


NOTES:
To verify that alb-ingress-controller has started, run:

  kubectl --namespace=kube-system get pods -l "app=alb-ingress-controller,component=controller,release=alb-ingress"

An example Ingress that makes use of the controller:

  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: alb
      alb.ingress.kubernetes.io/security-groups: sg-723a380a,sg-a6181ede,sg-a5181edd
      alb.ingress.kubernetes.io/subnets: subnet-a4f0098e,subnet-457ed533,subnet-95c904cd
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - path: /
              backend:
                serviceName: exampleService
                servicePort: 80


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

## 1-2. Option 2 install with Kubectl
### 1-2-1. Download aws-alb-ingress-controller to local
```
$ mkdir environment
$ cd ./environment
$ git clone --depth=1 --branch=master https://github.com/kubernetes-sigs/aws-alb-ingress-controller.git && rm -rf aws-alb-ingress-controller/.git
```

### 1-2-2. Edit alb-ingress-controller.yaml
*cluster-name
*AWS_ACCESS_KEY_ID
*AWS_SECRET_ACCESS_KEY

### 1-2-3.
```
$ kubectl apply -f aws-alb-ingress-controller/docs/examples/rbac-role.yaml
```

****
```
$ kubectl get clusterrolebinding -n=kube-system
NAME                                                   AGE
alb-ingress-controller                                 70s
```

```
$ kubectl get clusterrole -n=kube-system
NAME                                                   AGE
alb-ingress-controller                                                 99s
```

```
$ kubectl get serviceaccount -n=kube-system
NAME                   SECRETS   AGE
alb-ingress            1         5m
```

### 1-2-4.
```
$ kubectl apply -f aws-alb-ingress-controller/docs/examples/alb-ingress-controller.yaml
```

```
$ kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o alb-ingress[a-zA-Z0-9-]+)

```

```
$ kubectl delete -f aws-alb-ingress-controller/docs/examples/alb-ingress-controller.yaml
```

# 2. Deploy workload resource
## 2-1. Download the echoserver example to local
```
$ mkdir environment
$ cd ./environment
$ git clone --depth=1 --branch=master https://github.com/kubernetes-sigs/aws-alb-ingress-controller.git && rm -rf aws-alb-ingress-controller/.git
```

## 2-2. Deploy echoserver namespace, service and deployment
```
$ kubectl apply -f aws-alb-ingress-controller/docs/examples/echoservice/echoserver-namespace.yaml
$ kubectl apply -f aws-alb-ingress-controller/docs/examples/echoservice/echoserver-service.yaml
$ kubectl apply -f aws-alb-ingress-controller/docs/examples/echoservice/echoserver-deployment.yaml
```

**Check the namespace**
```
$ kubectl get namespaces
NAME          STATUS   AGE
default       Active   5d
echoserver    Active   7m
kube-public   Active   5d
kube-system   Active   5d
```

**Check the deployments and services**
```
$ kubectl get deploy,svc -n echoserver
NAME                               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/echoserver   1         1         1            1           6m

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/echoserver   ClusterIP   10.100.163.143   <none>        80/TCP    7m
```

## 2-3. Deploy ingress for echoserver
## 2-3-1. Edit ingress yaml
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: echoserver
  namespace: echoserver
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/tags: Environment=dev,Team=test
    alb.ingress.kubernetes.io/subnets: subnet-0429e5dd6d1affabb,subnet-076803b7f3e01846c,subnet-05f31c4f22068df24
spec:
  rules:
    - http:
        paths:
          - path: /
            backend:
              serviceName: echoserver
              servicePort: 80
```


### 2-3-2. Ingress deployment
```
$ kubectl apply -f aws-alb-ingress-controller/docs/examples/echoservice/echoserver-ingress.yaml
```

**Verification**
```
$ kubectl get deploy,svc,ingress -n echoserver
```

```
$ kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o 'alb-ingress[a-zA-Z0-9-]+') | grep 'echoserver\/echoserver'

```




# 3. Clean-up
## 3-1.
```
$ kubectl delete namespace echoserver
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



----------------

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: "2048-ingress"
  namespace: "2048-game"
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/subnets: subnet-0429e5dd6d1affabb,subnet-076803b7f3e01846c,subnet-05f31c4f22068df24
  labels:
    app: 2048-ingress
spec:
  rules:
    - http:
        paths:
          - path: /*
            backend:
              serviceName: "service-2048"
              servicePort: 80

```

```
$ kubectl apply -f aws-alb-ingress-controller/docs/examples/2048/2048-namespace.yaml
$ kubectl apply -f aws-alb-ingress-controller/docs/examples/2048/2048-deployment.yaml
$ kubectl apply -f aws-alb-ingress-controller/docs/examples/2048/2048-service.yaml
```

```
$ kubectl get deploy,svc -n 2048-game
```

```
$ kubectl apply -f aws-alb-ingress-controller/docs/examples/2048/2048-ingress.yaml
```

```
$ kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o 'alb-ingress[a-zA-Z0-9-]+')
```
-----

[Kubernetes Ingress with AWS ALB Ingress Controller](https://aws.amazon.com/blogs/opensource/kubernetes-ingress-aws-alb-ingress-controller/)