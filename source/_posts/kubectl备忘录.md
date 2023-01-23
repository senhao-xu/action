---
title: kubectl
date: 2022-12-03 13:18:34
tags: kubernetes
categories: kubernetes
description: kubectl备忘录.
---

## kubectl备忘录

### kubectl

```bash
#kubectl自动补全
yum -y install bash-completion && source /usr/share/bash-completion/bash_completion && source <(kubectl completion bash) && echo "source <(kubectl completion bash)" >> ~/.bashrc

#删除default命名空间下所有job
kubectl get job | awk '{print $1}' | xargs kubectl delete job

#删除pie-engine-infra命名空间下pod状态为ImagePullBackOff的资源
kubectl get pod -n pie-engine-infra | grep ImagePullBackOff | awk '{print $1}' | xargs kubectl delete pod -n pie-engine-infra
```
