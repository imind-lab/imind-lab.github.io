---
title: 在Linux上安装zookeeper集群
date: 2021-05-25 14:38:51
tags:
  - zookeeper
  - linux
---

##### 1、安装JDK1.8

##### 2、下载 Zookeeper

进入 https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/ 下载合适的版本，准备安装。
注意：这里需要下载 Linux 版本。这里以apache-zookeeper-3.6.2-bin.tar.gz为例

##### 3、解压 Zookeeper

把下载的文件 apache-zookeeper-3.6.2-bin.tar.gz 并放在/opt/modules 目录下。

```
tar -zxvf apache-zookeeper-3.6.2-bin.tar.gz /opt/modules/
cd /opt/modules/
mv apache-zookeeper-3.6.2-bin zookeeper-3.6.2
```

##### 4、修改zoo.cfg配置

```
cd /opt/modules/zookeeper-3.6.2/conf
cp zoo_sample.cfg zoo.cfg

mkdir /opt/modules/zookeeper-3.6.2/data
mkdir /opt/modules/zookeeper-3.6.2/logs

vi zoo.cfg

dataDir=/opt/modules/zookeeper-3.6.2/data
maxClientCnxns=1024
autopurge.snapRetainCount=3
autopurge.purgeInterval=1
# 在尾部添加
server.1=hadoop01:2888:3888
server.2=hadoop02:2888:3888
server.3=hadoop03:2888:3888
```

##### 5、添加myid文件

```
echo 1 > /opt/modules/zookeeper-3.6.2/data/myid
```

##### 6、修改服务器系统环境变量

```
vi /etc/profile.d/env.sh

#添加如下配置
#ZooKeeper
export ZK_HOME=/opt/modules/zookeeper-3.6.2
export PATH=$PATH:$ZK_HOME/bin

#配置生效
source /etc/profile.d/env.sh
```

##### 7、将zookeeper同步到其他服务器

```
xsync /opt/modules/zookeeper-3.6.2
xsync /etc/profile.d/env.sh

#hadoop02上执行
echo 2 > /opt/modules/zookeeper-3.6.2/data/myid
source /etc/profile.d/env.sh

#hadoop03上执行
echo 3 > /opt/modules/zookeeper-3.6.2/data/myid
source /etc/profile.d/env.sh
```

