# Workload via Workload via Application Load Balancer Ingress

# 1. Deploy ALB Ingress Controller
## 1-1. Download aws-alb-ingress-controller to local
```
$ mkdir environment
$ cd ./environment
$ git clone --depth=1 --branch=master https://github.com/kubernetes-sigs/aws-alb-ingress-controller.git && rm -rf aws-alb-ingress-controller/.git
$ cd ..
```

## 1-2. Deploy RBAC(Role-Based Access Control)
```
$ kubectl apply -f environment/aws-alb-ingress-controller/docs/examples/rbac-role.yaml
```

**Check ClusterRoleBinding**
```
$ kubectl get clusterrolebinding -n=kube-system
NAME                                                   AGE
alb-ingress-controller                                 70s
```

**Check ClusterRole**
```
$ kubectl get clusterrole -n=kube-system
NAME                                                   AGE
alb-ingress-controller                                                 99s
```

**Check ServiceAccount**
```
$ kubectl get serviceaccount -n=kube-system
NAME                   SECRETS   AGE
alb-ingress            1         5m
```

## 1-3. Edit alb-ingress-controller.yaml
**environment/aws-alb-ingress-controller/docs/examples/alb-ingress-controller.yaml**

* --ingress-class=alb
* --cluster-name=eks-workshop

## 1-4. Deploy pod alb-ingress-controller to namespace kube-system
```
$ kubectl apply -f environment/aws-alb-ingress-controller/docs/examples/alb-ingress-controller.yaml
```

```
$ kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o alb-ingress[a-zA-Z0-9-]+)
```

## 1-5. Create and Attach Instance Role Policy
### 1-5-1. Create AlbIngressControllerPolicy
```
$ aws iam create-policy --policy-name AlbIngressControllerPolicy --policy-document file://environment/aws-alb-ingress-controller/docs/examples/iam-policy.json
```

### 1-5-2. Attach the policy to the Node Instance Role
**The Node Instance Role name can be found from the Cloudformation Stack**
```
$ aws iam attach-role-policy --role-name eksctl-eks-workshop-nodegroup-0-NodeInstanceRole-<your_node_instance_role> --policy-arn arn:aws:iam::<your_account_id>:policy/AlbIngressControllerPolicy
```

# 2. Deploy workload resource - sample echoserver
## 2-1. Deploy echoserver namespace, service and deployment
```
$ kubectl apply -f environment/aws-alb-ingress-controller/docs/examples/echoservice/echoserver-namespace.yaml
$ kubectl apply -f environment/aws-alb-ingress-controller/docs/examples/echoservice/echoserver-service.yaml
$ kubectl apply -f environment/aws-alb-ingress-controller/docs/examples/echoservice/echoserver-deployment.yaml
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

## 2-2. Deploy ingress for echoserver
### 2-2-1. Get public subnets
**cluster-id:eks-workshop, region:us-east-1 or us-east-2**
```
$ eksctl get cluster eks-workshop --region=us-east-1

$ eksctl get cluster eks-workshop --region=us-east-2
```


### 2-2-2. Edit ingress yaml
**environment/aws-alb-ingress-controller/docs/examples/echoservice/echoserver-ingress.yaml**
**Use the public subnets from 2-2-1**
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


### 2-2-3. Ingress deployment
```
$ kubectl apply -f environment/aws-alb-ingress-controller/docs/examples/echoservice/echoserver-ingress.yaml
```

**Verification**
```
$ kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o 'alb-ingress[a-zA-Z0-9-]+') | grep 'echoserver\/echoserver'
```

```
$ kubectl get ingress -n echoserver

NAME         HOSTS   ADDRESS                                                                 PORTS   AGE
echoserver   *       285c4ac9-echoserver-echose-2ad7-122290727.us-east-2.elb.amazonaws.com   80      3m
```

## 2-3. Check the demo app on browser
[URL_FROM_2-2-2](http://)


# 3. Deploy workload resource - sample 2048
## 3-1. Deploy 2048 namespace, service and deployment
```
$ kubectl apply -f environment/aws-alb-ingress-controller/docs/examples/2048/2048-namespace.yaml
$ kubectl apply -f environment/aws-alb-ingress-controller/docs/examples/2048/2048-deployment.yaml
$ kubectl apply -f environment/aws-alb-ingress-controller/docs/examples/2048/2048-service.yaml
```

**Check the namespace**
```
$ kubectl get namespaces
```

**Check the deployments and services**
```
$ kubectl get deploy,svc -n 2048-game
```

## 3-2. Deploy ingress for 2048
### 3-2-1. Get public subnets
**cluster-id:eks-workshop, region:us-east-1 or us-east-2**
```
$ eksctl get cluster eks-workshop --region=us-east-1

$ eksctl get cluster eks-workshop --region=us-east-2
```

## 3-2-2. Edit ingress yaml
**environment/aws-alb-ingress-controller/docs/examples/2048/2048-ingress.yaml**
**Use the public subnets from 3-2-1**
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


### 3-2-3. Ingress deployment
```
$ kubectl apply -f environment/aws-alb-ingress-controller/docs/examples/2048/2048-ingress.yaml
```

**Verification**
```
$ kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o 'alb-ingress[a-zA-Z0-9-]+') | grep '2048'
```

```
$ kubectl get ingress -n 2048-game
NAME           HOSTS   ADDRESS                                                                  PORTS   AGE
2048-ingress   *       285c4ac9-2048game-2048ingr-6fa0-1267004985.us-east-2.elb.amazonaws.com   80      3m
```

## 3-3. Check the demo app on browser
[URL_FROM_3-2-2](http://)

# 4. Clean-up
## 4-1. Remove echoserver
### 4-1-1. Remove echoserver ingress and ALB
```
$ kubectl delete -f environment/aws-alb-ingress-controller/docs/examples/echoservice/echoserver-ingress.yaml
```

### 4-1-2. Remove echoserver namespace
```
$ kubectl delete namespace echoserver
```

## 4-2. Remove 2048
### 4-2-1. Remove echoserver ingress and ALB
```
$ kubectl delete -f environment/aws-alb-ingress-controller/docs/examples/2048/2048-ingress.yaml
```

### 4-2-2. Remove 2048 namespace
```
$ kubectl delete namespace 2048-game
```

## 4-3. Remove alb-ingress-controller related resources

```
$ kubectl delete -f environment/aws-alb-ingress-controller/docs/examples/alb-ingress-controller.yaml
```

```
$ kubectl delete -f environment/aws-alb-ingress-controller/docs/examples/rbac-role.yaml
```

