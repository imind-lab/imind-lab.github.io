---
title: K8s常用命令
date: 2021-05-23 06:58:51
tags:
  - kubernetes
categories:
  - kubernetes
---

###### 1、将当前上下文保存所有后续kubectl命令的namespace

```shell
$ kubectl config set-context $(kubectl config current-context) —namespace=micro
```

###### 2、列出所有namespace中具有状态的所有Pod

```shell
$ kubectl get pods --all-namespaces
```

###### 3、列出指定namespace中具有状态的所有Pod

```shell
$ kubectl get po -o wide -n <namspace1> -n <namespace2> -n <namespace3>
```

###### 4、显示当前默认的namespace

```shell
$ kubectl config view --minify | grep namespace
```

###### 5、k8s常用的aliases别名

```shell
alias k='kubectl'
alias kc='kubectl config view --minify | grep name'
alias kdp='kubectl describe pod'
alias krh='kubectl run --help | more'
alias ugh='kubectl get --help | more'
alias c='clear'
alias ke='kubectl explain'
alias kf='kubectl create -f'
alias kg='kubectl get pods --show-labels'
alias kr='kubectl replace -f'
alias kh='kubectl --help | more'
alias krh='kubectl run --help | more'
alias ks='kubectl get namespaces'
alias kga='k get pod --all-namespaces'
alias kgaa='kubectl get all --show-labels'

```

##### 6、VI配置，便于使用vi编辑YAML

创建 `~/.vimrc` 并添加以下内容

```shell
set smarttab
set expandtab
set shiftwidth=4
set tabstop=4
set number
```

##### 7、从kubectl命令创建YAML模板文件

```shell
kubectl run busybox --image=busybox --dry-run=client -o yaml --restart=Never > yamlfile.yaml

kubectl create job my-job --dry-run=client -o yaml --image=busybox -- date > yamlfile.yaml

kubectl get -o yaml deploy/nginx > 1.yaml (Ensure that you have a deployment named as nginx)

kubectl run busybox --image=busybox --dry-run=client -o yaml --restart=Never -- /bin/sh -c "while true; do echo hello; echo hello again;done" > yamlfile.yaml

kubectl run wordpress --image=wordpress –-expose –-port=8989 --restart=Never -o yaml

kubectl run test --image=busybox --restart=Never --dry-run=client -o yaml -- bin/sh -c 'echo test;sleep 100' > yamlfile.yaml （最后的增加 --bin 。这将创建yaml文件。）
```

创建YAML文件的另一个好办法是使用`wget` 命令直接从Internet获得文件

###### 8、使用kubectx和kubens分别管理上下文和namespace

https://github.com/ahmetb/kubectx

