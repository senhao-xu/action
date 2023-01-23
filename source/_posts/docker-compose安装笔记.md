---
title: docker-compose安装笔记
date: 2022-04-10 13:18:34
tags: docker
categories: docker
description: docker-compose安装笔记.
---
## 安装docker-compose

### 下载文件

#### github(国外)

``` bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && sudo chmod +x /usr/local/bin/docker-compose
```

#### daocloud(国内)

``` bash
curl -L https://get.daocloud.io/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

### 授予权限

``` bash
chmod +x /usr/local/bin/docker-compose
```

### 查看是否安装完成

``` bash
docker-compose --version
```

## 安装zookeeper

### 安装高可用zk

> docker-compose.yml

``` yaml
version: '3.1'
services:
  zk1:
    image: zookeeper
    restart: always
    container_name: zk1
    ports:
      - 2181:2181
    volumes:
      - /usr/local/docker/zookeeper/zk1/data:/data
      - /usr/local/docker/zookeeper/zk1/datalog:/datalog
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=zk1:2888:3888;2181 server.2=zk2:2888:3888;2181 server.3=zk3:2888:3888;2181
  zk2:
    image: zookeeper
    restart: always
    container_name: zk2
    ports:
      - 2182:2181
    volumes:
      - /usr/local/docker/zookeeper/zk2/data:/data
      - /usr/local/docker/zookeeper/zk2/datalog:/datalog
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zk1:2888:3888;2181 server.2=zk2:2888:3888;2181 server.3=zk3:2888:3888;2181
  zk3:
    image: zookeeper
    restart: always
    container_name: zk3
    ports:
      - 2183:2181
    volumes:
      - /usr/local/docker/zookeeper/zk3/data:/dada
      - /usr/local/docker/zookeeper/zk3/datalog:/datalog

    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zk1:2888:3888;2181 server.2=zk2:2888:3888;2181 server.3=zk3:2888:3888;2181
```

## 安装Mysql

### 可用MySQL

> docker-compose.yml

``` yaml
version: '3'
services:
  mysql-slave-lb:
    image: nginx:alpine
    container_name: mysql-slave-lb
    ports:
    - 30306:30306
    volumes:
    - /docker/nginx/nginx.conf:/etc/nginx/nginx.conf
    networks:
    - mysql
    depends_on:
    - mysql-master
    - mysql-slave1
    - mysql-slave2
  mysql-master:
    image: mysql:8.0.25
    container_name: mysql-master
    environment:
      MYSQL_ROOT_PASSWORD: "piesat"
      MASTER_SYNC_USER: "sync_admin" #设置脚本中定义的用于同步的账号
      MASTER_SYNC_PASSWORD: "piesat" #设置脚本中定义的用于同步的密码
      ADMIN_USER: "root" #当前容器用于拥有创建账号功能的数据库账号
      ADMIN_PASSWORD: "piesat"
      ALLOW_HOST: "10.10.%.%" #允许同步账号的host地址
      TZ: "Asia/Shanghai" #解决时区问题
    ports:
    - 3306:3306
    networks:
      mysql:
        ipv4_address: "10.10.10.10" #固定ip，因为从库在连接master的时候，需要设置host
    volumes:
    - /docker/mysql/init/master:/docker-entrypoint-initdb.d #挂载master脚本
    command:
    -  "--server-id=1"
    -  "--character-set-server=utf8mb4"
    -  "--collation-server=utf8mb4_unicode_ci"
    -  "--log-bin=mysql-bin"
    -  "--sync_binlog=1"
  mysql-slave1:
    image: mysql:8.0.25
    container_name: mysql-slave1
    environment:
      MYSQL_ROOT_PASSWORD: "piesat"
      SLAVE_SYNC_USER: "sync_admin" #用于同步的账号，由master创建
      SLAVE_SYNC_PASSWORD: "piesat"
      ADMIN_USER: "root"
      ADMIN_PASSWORD: "piesat"
      MASTER_HOST: "10.10.10.10" #master地址，开启主从同步需要连接master
      TZ: "Asia/Shanghai" #设置时区
    networks:
     mysql:
       ipv4_address: "10.10.10.20" #固定ip
    volumes:
    - /docker/mysql/slave:/docker-entrypoint-initdb.d #挂载slave脚本
    command:
    -  "--server-id=2"
    -  "--character-set-server=utf8mb4"
    -  "--collation-server=utf8mb4_unicode_ci"
  mysql-slave2:
    image: mysql:8.0.25
    container_name: mysql-slave2
    environment:
      MYSQL_ROOT_PASSWORD: "piesat"
      SLAVE_SYNC_USER: "sync_admin"
      SLAVE_SYNC_PASSWORD: "piesat"
      ADMIN_USER: "root"
      ADMIN_PASSWORD: "piesat"
      MASTER_HOST: "10.10.10.10"
      TZ: "Asia/Shanghai"
    networks:
      mysql:
        ipv4_address: "10.10.10.30" #固定ip
    volumes:
    - /docker/mysql/slave:/docker-entrypoint-initdb.d
    command: #这里需要修改server-id，保证每个mysql容器的server-id都不一样
    -  "--server-id=3"
    -  "--character-set-server=utf8mb4"
    -  "--collation-server=utf8mb4_unicode_ci"
networks:
  mysql:
    driver: bridge
    ipam:
      driver: default
      config:
      - subnet: "10.10.0.0/16"
```

> nginx.conf

``` bash
cat > nginx.conf <<EOF
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}

# 添加stream模块，实现tcp反向代理
stream {
    proxy_timeout 30m;
    upstream mysql-slave-cluster{
      #docker-compose.yml里面会配置固定mysql-slave的ip地址，这里就填写固定的ip地址
      server 10.10.10.20:3306 weight=1;
      server 10.10.10.30:3306 weight=1 backup; #备用数据库，当上面的数据库挂掉之后，才会使用此数据库，也就是如果上面的数据库没有挂，则所有的流量都很转发到上面的主库
    }
    server {
        listen 30306;
        proxy_pass mysql-slave-cluster;
    }
}
EOF
```