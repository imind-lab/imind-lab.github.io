---
title: 在Kubernetes上安装配置Traefik
date: 2021-05-25 14:38:51
tags:
  - kubernetes
  - traefik
---

#### 一、创建Traefik的自定义资源(CRD)文件

traefik-crd.yaml 

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: infra

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: ingressroutes.traefik.containo.us
spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: IngressRoute
    plural: ingressroutes
    singular: ingressroute
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: middlewares.traefik.containo.us
spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: Middleware
    plural: middlewares
    singular: middleware
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: ingressroutetcps.traefik.containo.us
spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: IngressRouteTCP
    plural: ingressroutetcps
    singular: ingressroutetcp
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: ingressrouteudps.traefik.containo.us
spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: IngressRouteUDP
    plural: ingressrouteudps
    singular: ingressrouteudp
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: tlsoptions.traefik.containo.us
spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: TLSOption
    plural: tlsoptions
    singular: tlsoption
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: tlsstores.traefik.containo.us
spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: TLSStore
    plural: tlsstores
    singular: tlsstore
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: traefikservices.traefik.containo.us
spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: TraefikService
    plural: traefikservices
    singular: traefikservice
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: serverstransports.traefik.containo.us
spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: ServersTransport
    plural: serverstransports
    singular: serverstransport
  scope: Namespaced

---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: traefik-ingress-controller
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
      - networking.k8s.io
    resources:
      - ingresses
      - ingressclasses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses/status
    verbs:
      - update
  - apiGroups:
      - traefik.containo.us
    resources:
      - middlewares
      - ingressroutes
      - traefikservices
      - ingressroutetcps
      - ingressrouteudps
      - tlsoptions
      - tlsstores
      - serverstransports
    verbs:
      - get
      - list
      - watch

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
  - kind: ServiceAccount
    name: traefik-ingress-controller
    namespace: infra

```

创建Traefik自定义资源

```
kubectl apply -f traefik-crd.yaml
```

#### 二、创建Taefik Deployment部署

traefik-deployment.yaml

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  namespace: infra
  name: traefik-ingress-controller

---
kind: Deployment
apiVersion: apps/v1
metadata:
  namespace: infra
  name: traefik
  labels:
    app: traefik

spec:
  replicas: 1
  selector:
    matchLabels:
      app: traefik
  template:
    metadata:
      labels:
        app: traefik
    spec:
      serviceAccountName: traefik-ingress-controller
      containers:
        - name: traefik
          image: traefik:v2.4
          args:
            - --accesslog
            - --entrypoints.web.Address=:8000
            - --entrypoints.websecure.Address=:4443
            - --entrypoints.mysql.Address=:3306
            - --providers.kubernetescrd
            - --certificatesresolvers.myresolver.acme.tlschallenge
            - --certificatesresolvers.myresolver.acme.email=foo@you.com
            - --certificatesresolvers.myresolver.acme.storage=acme.json
            # Please note that this is the staging Let's Encrypt server.
            # Once you get things working, you should remove that whole line altogether.
            - --certificatesresolvers.myresolver.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory
          ports:
            - name: web
              containerPort: 8000
            - name: websecure
              containerPort: 4443
            - name: admin
              containerPort: 8080
            - name: mysql
              containerPort: 3306
```

创建Traefik部署

```
kubectl apply -f traefik-deployment.yaml
```

#### 三、创建Traefik 服务

traefik-svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: traefik
  namespace: infra
spec:
  type: NodePort
  ports:
    - protocol: TCP
      name: web
      port: 8000
      nodePort: 80
    - protocol: TCP
      name: admin
      port: 8080
      nodePort: 8080
    - protocol: TCP
      name: websecure
      port: 443
      nodePort: 443
    - protocol: TCP
      name: mysql
      port: 3306
      nodePort: 3306
  selector:
    app: traefik
```

创建K8s Traefik服务

```shell
kubectl apply -f traefik-svc.yaml
```

错误提示：

The Service "traefik" is invalid: spec.ports[0].nodePort: Invalid value: 80: provided port is not in the valid range. The range of valid ports is 30000-32767

#### 四、更改Kubernetes服务节点端口范围：

1）登录Docker VM:

```shell
docker run --rm -it --privileged --pid=host walkerlee/nsenter -t 1 -m -u -i -n sh
```

2）编辑kube-apiserver.yaml

```shell
 vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

3）在Pod spec中添加

```yaml
spec:
  containers:
  - command:
    - kube-apiserver
    ...
    - --service-cluster-ip-range=10.96.0.0/12
    - --service-node-port-range=1-65535   #添加本行
    - --tls-cert-file=/run/config/pki/apiserver.crt
    ...
```

4）保存并退出

5）重启Kubernetes



创建Traefik服务

```shell
kubectl apply -f traefik-svc.yaml
```

访问Traefik WebUI

http://lk.julive.com:8080

Traefik安装配置成功



