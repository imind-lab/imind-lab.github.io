---
title: Linux上Flume安装配置
date: 2021-05-25 14:38:51
tags:
  - flume
  - linux
categories:
  - 大数据
---

##### 1、下载
```shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/flume/1.9.0/apache-flume-1.9.0-bin.tar.gz
```

##### 2、解压
```shell
tar zxvf apache-flume-1.9.0-bin.tar.gz -C /opt/modules
```

##### 3、改名
```shell
mv apache-flume-1.9.0-bin flume-1.9.0
```

##### 4、解决包冲突
```shell
cd /opt/modules/flume-1.9.0/lib

mv guava-11.0.2.jar guava-11.0.2.jar.bak
```

##### 5、设置环境变量
```shell
cd ../conf/

cp flume-env.sh.template flume-env.sh

vi flume-env.sh
```
导入java_home环境变量

