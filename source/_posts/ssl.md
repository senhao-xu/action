---
title: ssl
date: 2022-04-16 14:49:22
tags: Linux
categories: Linux
description: 申请ssl证书
---

## 申请免费ssl证书

GitHub 仓库地址:  [GitHub](https://github.com/acmesh-official/acme.sh)

中文文档：[说明 · acmesh-official/acme.sh Wiki (github.com)](https://github.com/acmesh-official/acme.sh/wiki/说明)

### 安装acme

```bash
curl https://get.acme.sh | sh -s email=my@example.com
```

OR:

```bash
wget -O -  https://get.acme.sh | sh -s email=my@example.com
```

### 安装socat

```bash
yum -y install socat
```

### 添加别名

```bash
alias acme.sh=~/.acme.sh/acme.sh
```

### 注册账号

```bash
acme.sh --register-account -m my@example.com
```

### 打开防火墙80端口

```bash
firewall-cmd --zone=public --add-port=80/tcp --permanent && firewall-cmd --reload
```

### 申请证书

####  ZeroSSL

```bash
acme.sh --set-default-ca --server zerossl --issue -d my@example.com  --standalone -k ec-256
```

#### letsencrypt

```bash
acme.sh --set-default-ca --server letsencrypt --issue -d my@example.com  --standalone -k ec-256
```

#### Buypass

```bash
acme.sh --set-default-ca --server buypass --issue -d my@example.com  --standalone -k ec-256
```

### 更新证书

目前证书在 60 天以后会自动更新, 你无需任何操作. 今后有可能会缩短这个时间, 不过都是自动的, 你不用关心.

请确保 cronjob 正确安装, 看起来是类似这样的:

```bash
crontab  -l

56 * * * * "/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh" > /dev/null
```

### 自签证书

#### 生成私钥

```bash
openssl ecparam -genkey -name prime256v1 -out ca.key
```

#### 生成证书

```bash
openssl req -new -x509 -days 36500 -key ca.key -out ca.crt  -subj "/CN=[aa.com]"
```

### 使用docker运行acme

#### 运行docker容器

执行以下命令运行acme程序，该命令使用腾讯云作为域名服务商，获取[DNSPod Token](https://console.dnspod.cn/account/token/token)地址.

```bash
docker run  -itd  -v "/root/acme":/acme.sh  -e DP_Key="[DNSPod Token]" -e DP_Id="[id]" --net=host --name=acme.sh neilpang/acme.sh daemon
```

#### 选择SSL服务商

```bash
docker exec acme.sh --set-default-ca --server letsencrypt
```

#### 注册账户

```bash
docker exec acme.sh --register-account -m qazwsxcc@gmail.com
```

#### 申请泛域名证书

```bash
docker exec acme.sh --issue --dns dns_dp -d [example.com] -d [*.example.com]
```

#### 安装证书

```bash
docker exec acme.sh --install-cert -d senhao.top \
--key-file       /acme.sh/senhao.top/senhao.top.key  \
--fullchain-file /acme.sh/senhao.top/fullchain.cer
```

### 本机运行acme

```bash
#安装acme
curl https://get.acme.sh | sh -s email=br5cyrqxt@tempmail.cn

#设置别名
alias acme.sh=~/.acme.sh/acme.sh

#配置SSL服务商
acme.sh --set-default-ca --server letsencrypt

#注册账户
acme.sh --register-account -m br5cyrqxt@tempmail.cn

#配置dns
export DP_Id="123123"
export DP_Key="123123123"

#创建证书
acme.sh --issue --dns dns_dp -d *.senhao.top -d senhao.top -w /home/acme/senhao

#查看证书
acme.sh --list

#删除指定证书
acme.sh --remove -d senhao.top
```

#### 安装证书

```bash
acme.sh --install-cert -d example.com \
--key-file       /path/to/keyfile/in/nginx/key.pem  \
--fullchain-file /path/to/fullchain/nginx/cert.pem \
--reloadcmd     "service nginx force-reload"
```

#### 自动更新

```bash
crontab  -l

56 * * * * "/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh" > /dev/null
```



