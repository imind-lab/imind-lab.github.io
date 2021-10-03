---
title: Mac下配置微服务集成开发环境
date: 2021-05-23 06:58:51
tags:
  - kubernetes
  - docker
categories:
  - kubernetes
---

#### 一、安装Docker-Desktop

##### 1、下载与安装

下载地址：https://hub.docker.com/editions/community/docker-ce-desktop-mac/

安装完成后，启动Docker，点击鲸鱼图标可显示docker的相关操作。

##### 2、配置镜像加速

在Docker的Preferences中配置加速器。

阿里云加速器地址：https://754jn7no.mirror.aliyuncs.com 。

#### 二、开启Docker内置Kubernetes

##### 1、拉取kubernetes系统镜像

国内网络不能下载 `Kubernetes` 集群所需要的镜像。 而GitHub Actions实现 `k8s.gcr.io` 上 `kubernetes` 依赖镜像自动同步到 Docker Hub上指定的仓库中。 通过k8s-docker-desktop-for-mac将所需镜像从 `Docker Hub` 的同步仓库中取回，并重新打上原始的`tag`. 

```shell
git clone https://github.com/maguowei/k8s-docker-desktop-for-mac.git
cd k8s-docker-desktop-for-mac
./load_images.sh
```

##### 2、开启kubernetes

在Docker的Preferences中开启enable kubernetes选项。

#### 三、安装Kubernetes提效工具

##### 1、命令行小工具：

下载地址：https://github.com/ahmetb/kubectx

```sh
mv kubectx_v0.9.3_darwin_x86_64/kubectx /usr/local/bin
mv kubens_v0.9.3_darwin_x86_64/kubens /usr/local/bin
```

##### 2、Kubernetes集群管理工具Kuboard的安装

1）安装

```sh
kubectl apply -f https://kuboard.cn/install-script/kuboard.yaml
kubectl apply -f https://addons.kuboard.cn/metrics-server/0.3.7/metrics-server.yaml
```

2）查看Kuboard运行状态

```shell
kubectl get pods -l k8s.kuboard.cn/name=kuboard -n kube-system
# NAME                       READY   STATUS        RESTARTS   AGE
# kuboard-54c9c4f6cb-6lf88   1/1     Running       0          45s
```

3）配置访问

traefik-kuboard.yaml

```yaml
kind: IngressRoute
metadata:
  name: ingresskuboard
  namespace: kube-system
spec:
  entryPoints:
    - web
  routes:
  - match: Host(`lka.imind.tech`)
    kind: Rule
    services:
    - name: kuboard
      port: 80
```

创建kuboard的traefik路由

```sh
kubectl apply -f traefik-kuboard.yaml
```

3）获取Token

```sh
echo $(kubectl -n kube-system get secret $(kubectl -n kube-system get secret | grep kuboard-user | awk '{print $1}') -o go-template='{{.data.token}}' | base64 -d)
```

4)  添加本地hosts文件

```shell
echo '127.0.0.1	lka.imind.tech' > /etc/hosts
```

5）使用输出信息中 token字段访问Kuboard

访问kuboard

http://lka.imind.tech

##### 
