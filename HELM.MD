# 1. Helm Initialization

## 1-1. Install Helm CLI
```
brew install kubernetes-helm
```

## 1-2. Install Helm Service Account
```
$ kubectl create -f helm/rbac-config.yaml
```

**Check the ServiceAccount and the ClusterRoleBinding**
```
$ kubectl get clusterrolebinding -n=kube-system
$ kubectl get serviceaccount -n=kube-system
```

**RBAC means Role-based Access Control**
[RBAC](https://docs.helm.sh/using_helm/#role-based-access-control)

## 1-3. Install Tiller Server
```
$ helm init --service-account tiller
```

**Check the tiller server pod**
```
$kubectl get pod -n kube-system
NAME                                    READY   STATUS    RESTARTS   AGE
tiller-deploy-778f674bf5-vm9pz          1/1     Running   0          1s
```

## 1-4. Install the Helm Registry Plugin
```
$ mkdir -p ~/.helm/plugins/
$ cd ~/.helm/plugins/ && git clone https://github.com/app-registry/appr-helm-plugin.git registry
```

**On first use it downloads required assets**
```
$ helm registry --help
Registry plugin assets do not exist, download them now !
downloading https://github.com/app-registry/appr-cli/releases/download/v0.4.1/cnr-linux-x64 ...
```