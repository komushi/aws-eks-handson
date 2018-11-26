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
TODO
### 1-2-1. 

# 2. Deploy workload resource
## 2-1. 

----------------------
# Delete chart alb-ingress-controller-helm
```
$ helm delete --purge alb-ingress
```