---
title: Linux上Kafka集群搭建与配置
date: 2021-05-25 14:38:51
tags:
  - kafka
  - linux
---

#### 一、准备环境

##### 1、编辑各节点配置文件，添加节点间的映射关系。

```shell
vi /etc/hosts
172.16.50.2 hadoop01
172.16.50.3 hadoop02
172.16.50.4 hadoop03
```

##### 2、配置Java环境（每个节点）

有关【配置java环境方法】，请参考这里。

##### 3、搭建zookeeper集群

有关【搭建zookeeper集群】，请参考这里。

#### 二、搭建kafka集群

##### 1、下载kafka安装包，解压，配置kafka环境变量

有关【kafka安装包下载方法】，请参考这里。

本文下载的kafka版本是kafka_2.12-2.6.0.tgz，解压到指定一个目录（比如：/opt/modules），配置kafka环境变量，并使其生效。实现命令如下：

```shell
tar -zxvf kafka_2.12-2.6.0.tgz -C /opt/modules

vi /etc/profile.d/env.sh

#set kafka environment
export KAFKA_HOME=/opt/modules/kafka_2.12-2.6.0
export PATH=$PATH:$KAFKA_HOME/bin

source /etc/profile.d/env.sh
```

##### 2、编辑配置文件

```shell
vi /opt/modules/kafka_2.12-2.6.0/config/server.properties

broker.id=0
listeners=PLAINTEXT://172.16.50.2:9092
log.dirs=/opt/modules/kafka_2.12-2.6.0/logs
zookeeper.connect=hadoop01:2181,hadoop02:2181,hadoop03:2181
```

##### 3、将kafka同步到其他服务器

```shell
xsync /opt/modules/kafka_2.12-2.6.0
xsync /etc/profile.d/env.sh

#hadoop02上执行
vi /opt/modules/kafka_2.12-2.6.0/config/server.properties

broker.id=1
listeners=PLAINTEXT://172.16.50.3:9092
log.dirs=/opt/modules/kafka_2.12-2.6.0/logs
zookeeper.connect=hadoop01:2181,hadoop02:2181,hadoop03:2181

source /etc/profile.d/env.sh

#hadoop03上执行
vi /opt/modules/kafka_2.12-2.6.0/config/server.properties

broker.id=2
listeners=PLAINTEXT://172.16.50.4:9092
log.dirs=/opt/modules/kafka_2.12-2.6.0/logs
zookeeper.connect=hadoop01:2181,hadoop02:2181,hadoop03:2181

source /etc/profile.d/env.sh
```

