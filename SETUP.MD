# Basic Setup

# 1. Prerequsite
## 1-1. Clone this repo
```
$ git clone https://gitlab.com/komushi/aws-eks-handson.git
```

## 1-2. Install/Upgrade kubectl
```
$ brew install kubernetes-cli
```

## 1-3. Install ekscli
```
$ brew install weaveworks/tap/eksctl
```

## 1-4. Ensure the ELB Service Role exists
```
aws iam get-role --role-name "AWSServiceRoleForElasticLoadBalancing" || aws iam create-service-linked-role --aws-service-name "elasticloadbalancing.amazonaws.com"
```

# 2. Create an EKS cluster
## 2-1. Create a cluster by eksctl
**cluster-id:eks-workshop, region:us-east-1 or us-east-2**
```
$ eksctl create cluster --name=eks-workshop --nodes=3 --node-ami=auto --region=us-east-2

$ eksctl create cluster --name=eks-workshop --nodes=3 --node-ami=auto --region=us-east-1 --zones=us-east-1a,us-east-1b,us-east-1c
```

**Check ~/.kube/config after cluster creation**
```
$ cat  ~/.kube/config
```

**If no value retrieve it**
**cluster-id:eks-workshop, region:us-east-1 or us-east-2**
```
$ eksctl utils write-kubeconfig --name=eks-workshop --region=us-east-2

$ eksctl utils write-kubeconfig --name=eks-workshop --region=us-east-1
```

## 2-2. Test the cluster
```
$ kubectl get nodes
NAME                              STATUS   ROLES    AGE   VERSION
ip-192-168-127-144.ec2.internal   Ready    <none>   18m   v1.10.3
ip-192-168-128-244.ec2.internal   Ready    <none>   18m   v1.10.3
ip-192-168-212-229.ec2.internal   Ready    <none>   18m   v1.10.3
```

## 2-3. Check all the info from the cluster
**namespace: kube-system**
```
$ kubectl get all -n kube-system
```

**namespace: default**
```
$ kubectl get all -n default
```
