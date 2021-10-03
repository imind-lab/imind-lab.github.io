---
title: 在Deepin上安装Kubernetes
date: 2021-06-24 10:58:58
tags:
  - kubernetes
  - docker
  - deepin
categories:
  - kubernetes
---

#### 1、安装docker

```shell
sudo apt-get update && apt-get install -y apt-transport-https
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

#### 2、关闭swap

```
#临时关闭swap
sudo swapoff -a
# 永久关闭swap分区
sudo sed -i 's/.*swap.*/#&/' /etc/fstab
```

#### 3、添加k8s源

```
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
```

#### 4、导入k8s密钥并更新软件源，安装 kubeadm, kubelet 和 kubectl

```shell
curl -fsSL https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
sudo apt-get update
sudo apt-get install kubelet kubeadm kubectl
```

#### 5、设置阿里云镜像加速

```
cat <<EOF >/etc/docker/daemon.json
{
    "registry-mirrors": ["https://754jn7no.mirror.aliyuncs.com"]
}
EOF
```

#### 6、拉取镜像

```
# 从阿里云拉取镜像并转换tag
for  i  in  `kubeadm config images list`;  do
    imageName=${i#k8s.gcr.io/}
    docker pull registry.aliyuncs.com/google_containers/$imageName
    docker tag registry.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
    docker rmi registry.aliyuncs.com/google_containers/$imageName
done;

# coredns错误处理
docker pull registry.aliyuncs.com/google_containers/coredns:1.8.0
docker tag registry.aliyuncs.com/google_containers/coredns:1.8.0 k8s.gcr.io/coredns/coredns:v1.8.0
docker rmi registry.aliyuncs.com/google_containers/coredns:1.8.0
```

#### 7、kubeadm初始化

```shell
kubeadm init --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=NumCPU 
# --ignore-preflight-errors=NumCPU 忽略cpu错误,如果cpu核心数足够,可以不加.
# 必须要带上--pod-network-cidr=10.244.0.0/16，不然设置网络的时候会报错

# 如果初始化出错或者想重新初始化，可以使用如下命令
kubeadm reset

# 出现 Your Kubernetes master has initialized successfully!,安装成功
# 安装成功后执行
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

#### 8、安装网络插件

```shell
kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
```

#### 9、设置mater节点为可调度,因为默认情况下K8s的master节点是不能运行Pod

```shell
kubectl taint nodes --all node-role.kubernetes.io/master-
```

