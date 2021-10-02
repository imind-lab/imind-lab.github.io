---
title: Hadoop3.2完全分布式集群搭建方法（CentOS7.9+Hadoop3.2.1）
date: 2020-10-22 06:58:51
tags:
  - linux
  - hadoop
---

本文详细介绍搭建4个节点的完全分布式Hadoop集群的方法，Linux系统版本是CentOS 7.9，Hadoop版本是3.2.1，JDK版本是1.8。

#### 一、准备环境

##### 1、编辑各节点配置文件，添加主节点和从节点的映射关系。

```shell
vi /etc/hosts
172.16.50.2 hadoop01
172.16.50.3 hadoop02
172.16.50.4 hadoop03
```

2. ##### 关闭防火墙（每个节点）

```shell
systemctl stop firewalld #关闭服务

systemctl disable firewalld #关闭开机自启动
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
```

3. ##### 配置免密码登录

  有关【配置免密码登录方法】，请参考这里。
4. ##### 配置Java环境（每个节点）

  有关【配置java环境方法】，请参考这里。

#### 二、搭建Hadoop完全分布式集群

在各个节点上安装与配置Hadoop的过程都基本相同，因此可以在每个节点上安装好Hadoop后，在主节点hadoop01上进行统一配置，然后通过scp 命令将修改的配置文件拷贝到各个从节点上即可。

##### 1、下载Hadoop安装包，解压，配置Hadoop环境变量

本文下载的Hadoop版本是3.2.1，解压到指定目录（比如：/opt/modules），解压到指定目录，配置Hadoop环境变量，并使其生效。实现命令如下：

```shell
tar -zxvf hadoop-3.2.1.tar.gz -C /opt/modules #解压到/opt/modules目录

vi /etc/profile.d/env.sh #配置Hadoop环境变量

#Hadoop
export HADOOP_HOME=/opt/modules/hadoop-3.2.1 # 该目录为解压安装目录
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop

source /etc/profile.d/env.sh
```

##### 2、配置Hadoop环境脚本文件中的JAVA_HOME参数

```shell
cd /opt/modules/hadoop-3.2.1/etc/hadoop #进入Hadoop安装目录下的etc/hadoop目录

#分别在hadoop-env.sh、mapred-env.sh、yarn-env.sh文件中添加或修改如下参数：
vi hadoop-env.sh 
export JAVA_HOME=/usr/local/java/jdk1.8.0_201  # 路径为jdk安装路径
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```

##### 3、配置core-site.xml

```shell
vi /opt/modules/hadoop-3.2.1/etc/hadoop/core-site.xml
```

```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop1:9000</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/opt/modules/hadoop-3.2.1/tmp</value>
    <description>Abase for other temporary directories.</description>
  </property>
  <property>
   <name>hadoop.proxyuser.hadoop.hosts</name>
   <value>*</value>
  </property>
  <property>
   <name>hadoop.proxyuser.hadoop.groups</name>
   <value>*</value>
  </property>
</configuration>
```

##### 4、配置hdfs-site.xml

```shell
vi /opt/modules/hadoop-3.2.1/etc/hadoop/hdfs-site.xml
```

```xml
<configuration>
  <property>
    <name>dfs.namenode.http-address</name>
    <value>hadoop01:50070</value>
  </property>
  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>hadoop02:50090</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///opt/modules/hadoop-3.2.1/dfs/name</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///opt/modules/hadoop-3.2.1/dfs/data</value>
  </property>
  <property>
    <name>dfs.namenode.checkpoint.dir</name>
    <value>file:///opt/modules/hadoop-3.2.1/dfs/namesecondary</value>
    <description>Determines where on the local filesystem the DFSsecondary name node should store the temporary images to merge. If this is acomma-delimited list of directories then the image is replicated in all of thedirectories for redundancy.</description>
  </property>
  <property>
    <name>dfs.permissions.enabled</name>
    <value>false</value>
  </property>
  <property>
    <name>dfs.replication</name>
    <value>3</value>
  </property>
  <property>
    <name>dfs.webhdfs.enabled</name>
    <value>true</value>
  </property>
  <property>
    <name>dfs.permissions</name>
    <value>false</value>
    <description>配置为false后，可以允许不要检查权限就生成dfs上的文件，方便倒是方便了，但是你需要防止误删除.</description>
  </property>
  <property>
    <name>dfs.namenode.handler.count</name>
    <value>21</value>
  </property>
</configuration>
```

##### 5、配置mapred-site.xml

```shell
vi /opt/modules/hadoop-3.2.1/etc/hadoop/mapred-site.xml
```

```xml
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value> #设置MapReduce的运行平台为yarn
  </property>
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop01:10020</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop01:19888</value>
  </property>
  <property>
  	<name>yarn.app.mapreduce.am.env</name>
  	<value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
	</property>
  <property>
    <name>mapreduce.map.env</name>
    <value>HADOOP_MAPRED_HOME=${HADOP_HOME}</value>
  </property>
  <property>
    <name>mapreduce.reduce.env</name>
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
  </property>
  <property>
    <name>mapreduce.application.classpath</name>
    <value>
      ${HADOOP_HOME}/etc/hadoop,
      ${HADOOP_HOME}/share/hadoop/common/*,
      ${HADOOP_HOME}/share/hadoop/common/lib/*,
      ${HADOOP_HOME}/share/hadoop/hdfs/*,
      ${HADOOP_HOME}/share/hadoop/hdfs/lib/*,
      ${HADOOP_HOME}/share/hadoop/mapreduce/*,
      ${HADOOP_HOME}/share/hadoop/mapreduce/lib/*,
      ${HADOOP_HOME}/share/hadoop/yarn/*,
      ${HADOOP_HOME}/share/hadoop/yarn/lib/*
    </value>
  </property>
</configuration>
```

###### 6、配置yarn-site.xml

```shell
vi /opt/modules/hadoop-3.2.1/etc/hadoop/yarn-site.xml
```

```xml
<configuration>
  <property>
    <name>yarn.resourcemanager.hostname</name> #指定yarn的ResourceManager管理界面的地址，不配的话，Active Node始终为0
    <value>hadoop01</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services</name> #reducer获取数据的方式
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.resourcemanager.webapp.address</name>
    <value>hadoop01:8088</value>
    <description>配置外网只需要替换外网ip为真实ip，否则默认为 localhost:8088</description>
  </property>
  <property>
    <name>yarn.scheduler.maximum-allocation-mb</name>
    <value>1024</value>
    <description>每个节点可用内存,单位MB,默认8182MB</description>
  </property>
  <property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
    <discription>忽略虚拟内存的检查，如果你是安装在虚拟机上，这个配置很有用，配上去之后后续操作不容易出问题。</discription>
  </property>
  <property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
  </property>
  <property>
    <name>yarn.log.server.url</name>
    <value>http://hadoop01:19888/jobhistory/logs</value>
  </property>
  <property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>604800</value>
  </property>
</configuration>
```

##### 7、配置启动脚本，添加HDFS和Yarn权限

```shell
# 添加HDFS权限：编辑如下脚本，在第二行空白位置添加HDFS权限
vi sbin/start-dfs.sh 
vi sbin/stop-dfs.sh

HDFS_DATANODE_USER=root
HDFS_DATANODE_SECURE_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root

# 添加Yarn权限：编辑如下脚本，在第二行空白位置添加Yarn权限
vi sbin/start-yarn.sh 
vi sbin/stop-yarn.sh 

YARN_RESOURCEMANAGER_USER=root
HDFS_DATANODE_SECURE_USER=yarn
YARN_NODEMANAGER_USER=root
```

##### 8、初始化 & 启动

```shell
#格式化
bin/hdfs namenode -format

#启动（两种方式均可启动）
方法一：
sbin/start-all.sh

方法二：
sbin/start-dfs.sh 
sbin/start-yarn.sh

```

##### 9、添加从节点

```
vi /opt/modules/hadoop-3.2.1/etc/hadoop/workers

hadoop01
hadoop02
hadoop03
```

10. Web端口访问

```shel
# 查看防火墙状态
firewall-cmd --statex
# 临时关闭
systemctl stop firewalld
# 禁止开机启动
systemctl disable firewalld
```

在浏览器输入：http://hadoop01:8088打开ResourceManager页面。

在浏览器输入：http://hadoop01:50070打开Hadoop Namenode页面。