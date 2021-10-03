---
title: Hadoop集群配置免密SSH登录方法
date: 2020-05-25 14:38:51
tags:
  - hadoop
  - linux
categories:
  - 大数据
---

Hadoop集群包含1个主节点和2个从节点，需要实现各节点之间的免密码登录，下面介绍具体的实现方法。

#### 一、Hadoop集群环境

节点名称		节点IP

hadoop01		172.16.50.2
hadoop02		172.16.50.3
hadoop03		172.16.50.4

#### 二、免密登录原理

每台主机authorized_keys文件里面包含的主机（ssh密钥），该主机都能无密码登录，所以只要每台主机的authorized_keys文件里面都放入其他主机（需要无密码登录的主机）的ssh密钥就行了。

#### 三、实现方法

##### 1、配置每个节点的hosts文件

```shell
vim /etc/hosts
172.16.50.1 master
172.16.50.2 node01
172.16.50.3 node02
172.16.50.4 node03
```

##### 2、每个节点生成ssh密钥 

```shell
ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
.....................
```


​		执行命令后会在~目录下生成.ssh文件夹，里面包含id_rsa和id_rsa.pub两个文件。

注：使用ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa命令可避免上述交互式操作。

把本机的公钥追到其他节点root的 .ssh/authorized_keys 里
		ssh-copy-id root@hadoop01，输入密码确认
		ssh-copy-id root@hadoop02，输入密码确认
		ssh-copy-id root@hadoop03，输入密码确认

就可以免密登录了

其他机器同样执行以上步骤