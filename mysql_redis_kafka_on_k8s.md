---
title: 在Kubernetes上安装Mysql、Redis、Kafka
date: 2021-07-23 06:58:51
tags:
  - kubernetes
  - mysql
  - redis
  - kafka
---

##### 一、安装mysql

```shell
# 设置要安装命名空间
kubens infra

# 添加helm仓库
helm repo add bitnami https://charts.bitnami.com/bitnami

# 安装mysql
helm install mysql bitnami/mysql

#output
WARNING: This chart is deprecated
NAME: mysql
LAST DEPLOYED: Mon May 31 15:04:17 2021
NAMESPACE: infra
STATUS: deployed
REVISION: 1
NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
mysql.infra.svc.cluster.local

To get your root password run:

    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace infra mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

To connect to your database:

1. Run an Ubuntu pod that you can use as a client:

    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il

2. Install the mysql client:

    $ apt-get update && apt-get install mysql-client -y

3. Connect using the mysql cli, then provide your password:
    $ mysql -h mysql -p

To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306

    # Execute the following command to route the connection:
    kubectl port-forward svc/mysql 3306

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
```



配置MySQL代理网关

Traffic-mysql.yaml

```shel
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRouteTCP
metadata:
  name: ingressmysql
  namespace: infra
spec:
  entryPoints:
    - mysql
  routes:
  - match: HostSNI(`*`)
    services:
    - name: mysql
      port: 3306
```

创建mysql的traefik路由

```shell
kubectl apply -f traefik-mysql.yaml
```



##### 6、安装redis

```shell
# 安装Redis
helm install redis bitnami/redis

#output
NAME: redis
LAST DEPLOYED: Mon May 31 15:07:07 2021
NAMESPACE: infra
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

Redis(TM) can be accessed on the following DNS names from within your cluster:

    redis-master.infra.svc.cluster.local for read/write operations (port 6379)
    redis-replicas.infra.svc.cluster.local for read-only operations (port 6379)



To get your password run:

    export REDIS_PASSWORD=$(kubectl get secret --namespace infra redis -o jsonpath="{.data.redis-password}" | base64 --decode)

To connect to your Redis(TM) server:

1. Run a Redis(TM) pod that you can use as a client:

   kubectl run --namespace infra redis-client --restart='Never'  --env REDIS_PASSWORD=$REDIS_PASSWORD  --image docker.io/bitnami/redis:6.2.3-debian-10-r22 --command -- sleep infinity

   Use the following command to attach to the pod:

   kubectl exec --tty -i redis-client \
   --namespace infra -- bash

2. Connect using the Redis(TM) CLI:
   redis-cli -h redis-master -a $REDIS_PASSWORD
   redis-cli -h redis-replicas -a $REDIS_PASSWORD

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace infra svc/redis-master 6379:6379 &
    redis-cli -h 127.0.0.1 -p 6379 -a $REDIS_PASSWORD
```

##### 7、安装kafka

```shell
# 安装kafka
helm install kafka bitnami/kafka

# output
NAME: kafka
LAST DEPLOYED: Mon May 31 15:09:13 2021
NAMESPACE: infra
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

Kafka can be accessed by consumers via port 9092 on the following DNS name from within your cluster:

    kafka.infra.svc.cluster.local

Each Kafka broker can be accessed by producers via port 9092 on the following DNS name(s) from within your cluster:

    kafka-0.kafka-headless.infra.svc.cluster.local:9092

To create a pod that you can use as a Kafka client run the following commands:

    kubectl run kafka-client --restart='Never' --image docker.io/bitnami/kafka:2.8.0-debian-10-r27 --namespace infra --command -- sleep infinity
    kubectl exec --tty -i kafka-client --namespace infra -- bash

    PRODUCER:
        kafka-console-producer.sh \
            --broker-list kafka-0.kafka-headless.infra.svc.cluster.local:9092 \
            --topic test

    CONSUMER:
        kafka-console-consumer.sh \
            --bootstrap-server kafka.infra.svc.cluster.local:9092 \
            --topic sms_callback_aliyun \
            --from-beginning
```

##### 8、安装cmak

```shell
helm repo add cmak https://eshepelyuk.github.io/cmak-operator
```



