---
title: Docker安装Mysql
date: 2020-05-27 06:58:51
tags:
  - linux
  - java
---

##### 1、安装必要的一些系统工具

```shell
yum install -y yum-utils device-mapper-persistent-data \
    lvm2 bash-completion  
```

##### 2、添加软件源信息

```shell
yum-config-manager --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install docker -y
```

##### 3、设置docker开机启动

```shell
systemctl start docker
systemctl enable docker
```

##### 4、修改docker-ce配置

```shell
vi /etc/docker/daemon.json

{"registry-mirrors": ["https://7j4ic6a5.mirror.aliyuncs.com"]}
```

##### 5、启动容器

```shell
docker run -d --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=hx123456 -v /opt/modules/mysql/data:/var/lib/mysql mysql/mysql-server:5.7
```

