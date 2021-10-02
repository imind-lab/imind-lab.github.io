---
title: Linux上Java的安装与配置
date: 2020-10-22 06:58:51
tags:
  - linux
  - java
---

##### 1、下载 JDK

进入 Oracle 官方网站 下载合适的 JDK 版本，准备安装。
注意：这里需要下载 Linux 版本。这里以jdk-8u201-linux-x64.tar.gz为例，你下载的文件可能不是这个版本，这没关系，只要后缀(.tar.gz)一致即可。

##### 2、创建目录

​		在/usr/local目录下创建java目录

```shell
mkdir /usr/local/java
```

##### 3、解压 JDK

把下载的文件 jdk-8u201-linux-x64.tar.gz 放在/usr/local/java/目录下。

```shell
tar -zxvf jdk-8u201-linux-x64.tar.gz -C /usr/local/java/
```

##### 4、设置环境变量

```shell
vi /etc/profile.d/env.sh
```

添加如下内容并保存：

```shell
#Set Java Environment
export JAVA_HOME=/usr/local/java/jdk1.8.0_201    
export JRE_HOME=/usr/local/java/jdk1.8.0_201/jre     
export CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
```

注意：其中 JAVA_HOME， JRE_HOME 请根据自己的实际安装路径及 JDK 版本配置。

让修改生效：

```shell
source /etc/profile.d/env.sh
```

##### 5、测试

```shell
java -version
```

显示 java 版本信息，则说明 JDK 安装成功：

java version "1.8.0_201"

Java(TM) SE Runtime Environment (build 1.8.0_201-b09)

Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)