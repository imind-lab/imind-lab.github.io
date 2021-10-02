---
title: Linux上Hive 安装与配置
date: 2020-10-22 06:58:51
tags:
  - linux
  - hive
---

#### 一、Hive 安装

```shell
tar -zxvf apache-hive-3.1.2-bin.tar.gz -C /opt/modules/
cd /opt/modules/
mv apache-hive-3.1.2-bin hive-3.1.2

vi /etc/profile.d/env.sh
#Hive
export HIVE_HOME=/opt/modules/hive-3.1.2
export PATH=$PATH:$HIVE_HOME/bin

source /etc/profile.d/env.sh

cp mysql-connector-java-5.1.48.jar /opt/modules/hive-3.1.2/lib/
```

配置Metastore到Mysql

```shell
cd /opt/modules/hive-3.1.2/conf/
cp hive-default.xml.template hive-site.xml
vi hive-site.xml
```





```xml
<configuration>
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://hadoop03:3306/metastore?createDatabaseIfNotExist=true&amp;useSSL=false&amp;useUnicode=true&amp;characterEncoding=utf8mb4</value>
    <description>
      JDBC connect string for a JDBC metastore.
      To use SSL to encrypt/authenticate the connection, provide database-specific SSL flag in the connection URL.
      For example, jdbc:postgresql://myhost/db?ssl=true for postgres database.
    </description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
    <description>Driver class name for a JDBC metastore</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
    <description>username to use against metastore database</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>hx123456</value>
    <description>password to use against metastore database</description>
  </property>
  <property>
    <name>metastore.warehouse.dir</name>
    <value>/user/hive/warehouse</value>
    <description>hive default warehouse, if nessecory, change it</description>
  </property>
	<property>
    <name>metastore.schema.verification</name>
    <value>false</value>
    <description>
      Enforce metastore schema version consistency.
      True: Verify that version information stored in is compatible with one from Hive jars.  Also disable automatic
            schema migration attempt. Users are required to manually migrate schema after Hive upgrade which ensures
            proper metastore schema migration. (Default)
      False: Warn if the version information stored in metastore doesn't match with one from in Hive jars.
    </description>
  </property>
  <property>
    <name>hive.server2.thrift.port</name>
    <value>10000</value>
    <description>Port number of HiveServer2 Thrift interface when hive.server2.transport.mode is 'binary'.</description>
  </property>
  <property>
    <name>hive.server2.thrift.bind.host</name>
    <value>hadoop01</value>
    <description>Bind host on which to run the HiveServer2 Thrift service.</description>
  </property>
  <property>
    <name>hive.metastore.event.db.notification.api.auth</name>
    <value>false</value>
    <description>
      Should metastore do authorization against database notification related APIs such as get_next_notification.
      If set to true, then only the superusers in proxy settings have the permission
    </description>
  </property>
  <property>  
    <name>metastore.local</name>  
    <value>false</value>  
  </property> 
  <property>  
    <name>metastore.thrift.uris</name>  
    <value>thrift://hadoop01:9083</value>  
  </property> 
  <property>  
    <name>metastore.thrift.port</name>  
    <value>9083</value>  
  </property> 
  <property>
    <name>hive.cli.print.header</name>
    <value>true</value>
    <description>Whether to print the names of the columns in query output.</description>
  </property>
  <property>
    <name>hive.cli.print.current.db</name>
    <value>true</value>
    <description>Whether to include the current database in the Hive prompt.</description>
  </property>
</configuration>


```

