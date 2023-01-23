---
title: Etcd
date: 2022-11-11 12:58:05
tags: kubernetes
categories: kubernetes
description: etcd简介及安装.
---

# ETCD

## ETCD简介

etcd是由CoreOS团队发的一个分布式一致性的KV存储系统，可用于服务注册发现和共享配置，随着CoreOS和Kubernetes等项目在开源社区日益火热，它们项目中都用到的etcd组件作为一个高可用强一致性的服务发现存储仓库，渐渐为开发人员所关注。在云计算时代，如何让服务快速透明地接入到计算集群中，如何让共享配置信息快速被集群中的所有机器发现，更为重要的是，如何构建这样一套高可用、安全、易于部署以及响应快速的服务集群，已经成为了迫切需要解决的问题。

GitHub地址：[GitHub](https://github.com/etcd-io/etcd)

镜像仓库地址  [coreos/etcd](https://quay.io/repository/coreos/etcd?tab=tags&tag=latest)

官方文档：[Quickstart | etcd](https://etcd.io/docs/v3.5/quickstart/)

### 优点

etcd作为一个受到ZooKeeper与doozer启发而催生的项目，除了拥有与之类似的功能外，更专注于以下四点：

**简单：**安装配置简单，而且提供了 HTTP API 进行交互，使用也很简单 

**安全：**支持 SSL 证书验证 

**快速：**根据官方提供的 benchmark 数据，单实例支持每秒 2k+ 读操作

**可靠：**采用 raft 算法，实现分布式系统数据的可用性和一致性

## 安装ETCD

```
参数说明： 
● –data-dir 指定节点的数据存储目录，若不指定，则默认是当前目录。这些数据包括节点ID，集群ID，集群初始化配置，Snapshot文件，若未指 定–wal-dir，还会存储WAL文件 
● –wal-dir 指定节点的was文件存储目录，若指定了该参数，wal文件会和其他数据文件分开存储 
● –name 节点名称 
● –initial-advertise-peer-urls 告知集群其他节点的URL，tcp2380端口用于集群通信 
● –listen-peer-urls 监听URL，用于与其他节点通讯 
● –advertise-client-urls 告知客户端的URL, 也就是服务的URL，tcp2379端口用于监听客户端请求 
● –initial-cluster-token 集群的ID 
● –initial-cluster 集群中所有节点 
● –initial-cluster-state 集群状态，new为新创建集群，existing为已存在的集群
```

### kubernetes

参考地址：[传送门](https://zhuanlan.zhihu.com/p/373016748)

#### 部署etcd集群实例

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: etcd
  labels:
    component: etcd
spec:
  serviceName: etcd
  replicas: 3
  selector:
    matchLabels:
      component: etcd
  template:
    metadata:
      name: etcd
      labels:
        component: etcd
    spec:
      containers:
        - name: etcd
          image: quay.io/coreos/etcd:latest
          imagePullPolicy: IfNotPresent
          env:
            - name: CLUSTER_SIZE
              value: "3"
            - name: SET_NAME
              value: "etcd"
          command:
            - /bin/sh
            - -ecx
            - |
              IP=$(hostname -i)
              PEERS=""
              for i in $(seq 0 $((${CLUSTER_SIZE} - 1))); do
                  PEERS="${PEERS}${PEERS:+,}${SET_NAME}-${i}=http://${SET_NAME}-${i}.${SET_NAME}:2380"
              done
              exec etcd --name ${HOSTNAME} \
                --listen-peer-urls http://${IP}:2380 \
                --listen-client-urls http://${IP}:2379,http://127.0.0.1:2379 \
                --advertise-client-urls http://${HOSTNAME}.${SET_NAME}:2379 \
                --initial-advertise-peer-urls http://${HOSTNAME}.${SET_NAME}:2380 \
                --initial-cluster-token etcd-cluster-1 \
                --initial-cluster ${PEERS} \
                --initial-cluster-state new \
                --data-dir /var/run/etcd/default.etcd
          ports:
            - containerPort: 2379
              name: client
            - containerPort: 2380
              name: peer
          volumeMounts:
            - name: etcd-storage
              mountPath: /var/run/etcd/default.etcd
      volumes:
        - name: etcd-storage
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: etcd
  annotations:
    # Create endpoints also if the related pod isn't ready
    service.alpha.kubernetes.io/tolerate-unready-endpoints: "true"
spec:
  ports:
    - port: 2379
      name: client
    - port: 2380
      name: peer
  clusterIP: None
  selector:
    component: etcd
---
apiVersion: v1
kind: Service
metadata:
  name: etcd-client
spec:
  ports:
    - name: http
      nodePort: 30453
      port: 2379
      targetPort: 2379
      protocol: TCP
  type: NodePort
  selector:
    component: etcd
```

#### 部署WebUI

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: e3w-configmap
  labels:
    config: e3w.ini
data:
  e3w-config.default.ini: |
    [app]
    port=8080
    auth=false
    [etcd]
    root_key=root
    dir_value=
    addr=etcd-0.etcd.default.svc.cluster.local:2379,etcd-1.etcd.default.svc.cluster.local:2379,etcd-2.etcd.default.svc.cluster.local:2379
    username=
    password=
    cert_file=
    key_file=
    ca_file=
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: e3w-deployment
  labels:
    app: e3w
spec:
  replicas: 1
  selector:
    matchLabels:
      app: etcd-client-e3w
  template:
    metadata:
      labels:
        app: etcd-client-e3w
    spec:
      containers:
        - name: e3w-app-container
          image: soyking/e3w:latest
          imagePullPolicy: IfNotPresent
          ports:
            - name: e3w-server-port
              containerPort: 8080
          volumeMounts:
            - name: e3w-configmap-volume
              mountPath: /app/conf/config.default.ini
              subPath: config.default.ini
      volumes:
        - name: e3w-configmap-volume
          configMap:
            name: e3w-configmap
            items:
              - key: e3w-config.default.ini
                path: config.default.ini
---
kind: Service
apiVersion: v1
metadata:
  name: e3w-service
spec:
  type: NodePort
  selector:
    app: etcd-client-e3w
  ports:
    - protocol: TCP
      targetPort: e3w-server-port
      nodePort: 30081
      port: 80
```

WebUi部署界面：访问IP:PORT

![image-20221110143044365](https://oss.senhao.top/image/etcd/WebUi.png)



## ETCD命令

> 查看版本信息

```bash
etcdctl version
```

> 查看集群成员信息

```bash
etcdctl member list
```

> 查看集群状态

```bash
etcdctl cluster-health
```

## ETCD读写操作

### 写入

```bash
etcdctl put greeting "Hello, etcd"

/ # etcdctl put greeting "Hello, etcd"
No help topic for 'put'
/ # export ETCDCTL_API=3
/ # etcdctl put greeting "Hello, etcd"
OK
```

如遇到`No help topic for 'put'` 的错误信息，执行`export ETCDCTL_API=3`即可解决.

### 读取

```bash
/ # etcdctl get greeting
greeting
Hello, etcd
```
