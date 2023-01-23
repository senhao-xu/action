---
title: kubernetes笔记
date: 2022-04-10 13:18:34
tags: kubernetes
categories: kubernetes
description: kubernetes笔记.
---

## kubernetes命令

### node

#### 获取节点列表

```bash
kubectl get nodes
```

#### 修改节点ROLES

```bash
kubectl label node [node-name] node-role.kubernetes.io/worker=worker
```

#### 创建节点标签

```bash
kubectl lable node [env_role]=[dev]
```

#### 查看节点信息

```bash
kubectl describe node [node-name]
```

#### 查看节点资源

```bash
kubectl describe node |grep -E '((Name|Roles):\s{6,})|(\s+(memory|cpu)\s+[0-9]+\w{0,2}.+%\))'
```

#### 污点

污点值：

 NoSchedule : 一定不会被调度

 PreferNoSchedule : 尽量不会被调度

 NoExecute: 不会调度，并且还会驱逐pod到其他node

**添加污点**

```bash
kubectl taint node <node-name> <key>=<value>:<effect>
```

**删除污点**

```bash
kubectl taint node <node-name><key>[:<effect>]-
```

**去除master污点**

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

**查看节点污点信息**

```bash
kubectl describe node [node-name] |grep Taint
```

### Pod

```bash
# 创建pod
kubectl create deployment [pod-name] --image=[image]

# 普通获取
kubectl get pod

# 获取详细信息
kubectl get pod -o wide

# 获取pod详细信息
kubectl describe pod [pod-name]

# 按时间排序
kubectl get pods --sort-by=.metadata.creationTimestamp

# 查看集群正在运行的pod总数量
kubectl get pods --all-namespaces | grep Running | wc -l

# 获取指定标签
kubectl get pod 
```

#### 修改集群pod限制

参考地址：[传送门](https://blog.csdn.net/qq_43164571/article/details/113106127)

```bash
#修改10-kubeadm.conf文件
vim /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
添加：
	Environment="KUBELET_NODE_MAX_PODS=--max-pods=600"
	ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS $KUBELET_NODE_MAX_PODS
#重启
systemctl daemon-reload && systemctl restart kubelet
```

### svc

#### 修改NodePort端口限制

编辑 `vim /etc/kubernetes/manifests/kube-apiserver.yaml` 文件添加`- --service-node-port-range=20000-30000` 配置。

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=172.17.216.80
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --insecure-port=0
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    - --service-node-port-range=20000-30000
```

#### 重启资源

```bash
kubectl delete pod kube-apiserver-master -n kube-system
```

#### 注意

- `对于已经创建的NodePort类型的Service，您需要删除重新创建`
- `如果您的集群有多个 Master 节点，您需要逐个修改每个节点上的 /etc/kubernetes/manifests/kube-apiserver.yaml 文件，并重启 apiserver`

### secret

#### 添加secret

```bash
kubectl create secret docker-registry secret-harbor --namespace=default \
--docker-server=http://10.1.52.65 --docker-username=admin \
--docker-password=Harbor12345 --docker-email=xusenhao@piesat.cn
```

### yaml

#### pod

```bash
kubectl   create  deployment  nginx  --image=nginx   --dry-run  -o  yaml  >  deployment.yaml
```

service

```bash
kubectl expose  deployment  nginx  --port=80  --target-port=80  --type=NodePort --dry-run -o yaml > service.yaml
```

#### 处理状态处于Terminating的服务

强删

```bash
kubectl patch pvc data-nfs-server-provisioner-0 -p '{"metadata":{"finalizers": [null]}}' --type merge
```

#### 编辑资源

```bash
KUBE_EDITOR="vim" kuberctl edit pod test
```

#### 批量删除指定状态资源

```bash
kubectl get pod -n [pie-engine-infra] | grep [ImagePullBackOff] | awk '{print $1}' | xargs kubectl delete pod -n [pie-engine-infra]
```

### PVC

### pvc等待pv分配

参考地址 : [传送门](https://blog.csdn.net/weixin_45625174/article/details/123920122)

出现的错误信息：

waiting for a volume to be created, either by external provisioner “gxf-nfs-storage” or manually created by system administrator**

```bash
#修改
vim /etc/kubernetes/manifests/kube-apiserver.yaml
#在- --tls-private-key-file=/etc/kubernetes/pki/apiserver.key下面添加如下：
- --feature-gates=RemoveSelfLink=false
#执行
kubectl  apply -f /etc/kubernetes/manifests/kube-apiserver.yaml
```

## Helm安装

### 下载Helm

```bash
wget https://get.helm.sh/helm-v3.2.4-linux-amd64.tar.gz
```

### 解压

```bash
tar -zxvf helm-v3.2.4-linux-amd64.tar.gz
```

### 迁移文件

```bash
mv linux-amd64/helm /usr/local/bin/helm
```

### 验证安装

```bash
helm version
```

### 换源

```bash
helm repo add apphub https://apphub.aliyuncs.com
```

### 查看源

```bash
helm repo list
```

### 添加Harbor源

```bash
helm repo add pie-chart http://10.1.52.65/chartrepo/pie-chart --username=admin --password=Harbor12345
```

## 接入托管kubernetes

### 添加托管kubernetes

```bash
export KUBECONFIG=$KUBECONFIG:${CONFIG_PATH}
```

例：export KUBECONFIG=$HOME/.kube/config:/root/config

### 配置上下文

#### 列出所有上下文信息

```bash
kubectl config get-contexts
```

#### 查看当前上下文信息

```bash
kubectl config current-context
```

#### 更改上下文信息

```bash
kubectl config use-context ${CONTEXT_NAME}
```

例：kubectl config use-context kubernetes-admin@kubernetes

#### 更改上下文的元素

```bash
kubectl config set-context ${CONTEXT_NAME}|--current --${KEY}=${VALUE}
```

#### 生效

```bash
export KUBECONFIG=/root/config
```

## kubernetes更换ip

[传送门](https://blog.csdn.net/shelutai/article/details/122807511)

[传送门](https://www.qikqiak.com/post/how-to-change-k8s-node-ip/)

## kubernetes版本升级和降级

[传送门](https://blog.csdn.net/qq_38419276/article/details/120603533)

## kubeadm

导出初始化时默认配置

```bash
kubeadm config print init-defaults
```

