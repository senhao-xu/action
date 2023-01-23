---
title: Docker安装笔记
date: 2022-09-03 12:58:05
tags: docker
categories: docker
description: docker安装笔记.
---
## 安装Docker

### Docker安装

卸载旧版本

```shell
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

### 安装yum-utils软件包

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```

### 更新yum软件包索引

```
yum makecache fast
```

### 安装Docker Engine，CLI和Containerd软件包

```
yum install -y docker-ce docker-ce-cli containerd.io
```

### 启动docker

```
systemctl start docker
```

### 设置docker服务开机自启

```
systemctl enable docker
```

### 使用镜像加速器

#### 阿里云

> 创建文件夹

```
sudo mkdir -p /etc/docker
```

> 写入文件

```
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://cffh6cda.mirror.aliyuncs.com"]
}
EOF
```

> 重启Docker

```bash
systemctl daemon-reload && systemctl restart docker
```

### 一键安装脚本

```bash
curl -fsSL https://get.docker.com | sudo bash -s docker && sudo systemctl enable --now docker
```

## 卸载Dcoker

### 卸载Docker

> 卸载Docker Engine，CLI和Containerd软件包

```
yum remove docker-ce docker-ce-cli containerd.io
```

> 删除容器

```bash
rm -rf /var/lib/docker
rm -rf /var/lib/containerd
```

## Docker配置文件

参考地址：[传送门](https://blog.csdn.net/u010918487/article/details/106475785)

### daemon.json

完整的配置模板：

```json
{
  "authorization-plugins": [],
  "data-root": "",
  "dns": [],
  "dns-opts": [],
  "dns-search": [],
  "exec-opts": [],
  "exec-root": "",
  "experimental": false,
  "features": {},
  "storage-driver": "",
  "storage-opts": [],
  "labels": [],
  "live-restore": true,
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file":"5",
    "labels": "somelabel",
    "env": "os,customer"
  },
  "mtu": 0,
  "pidfile": "",
  "cluster-store": "",
  "cluster-store-opts": {},
  "cluster-advertise": "",
  "max-concurrent-downloads": 3,
  "max-concurrent-uploads": 5,
  "default-shm-size": "64M",
  "shutdown-timeout": 15,
  "debug": true,
  "hosts": [],
  "log-level": "",
  "tls": true,
  "tlsverify": true,
  "tlscacert": "",
  "tlscert": "",
  "tlskey": "",
  "swarm-default-advertise-addr": "",
  "api-cors-header": "",
  "selinux-enabled": false,
  "userns-remap": "",
  "group": "",
  "cgroup-parent": "",
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 64000,
      "Soft": 64000
    }
  },
  "init": false,
  "init-path": "/usr/libexec/docker-init",
  "ipv6": false,
  "iptables": false,
  "ip-forward": false,
  "ip-masq": false,
  "userland-proxy": false,
  "userland-proxy-path": "/usr/libexec/docker-proxy",
  "ip": "0.0.0.0",
  "bridge": "",
  "bip": "",
  "fixed-cidr": "",
  "fixed-cidr-v6": "",
  "default-gateway": "",
  "default-gateway-v6": "",
  "icc": false,
  "raw-logs": false,
  "allow-nondistributable-artifacts": [],
  "registry-mirrors": [],
  "seccomp-profile": "",
  "insecure-registries": [],
  "no-new-privileges": false,
  "default-runtime": "runc",
  "oom-score-adjust": -500,
  "node-generic-resources": ["NVIDIA-GPU=UUID1", "NVIDIA-GPU=UUID2"],
  "runtimes": {
    "cc-runtime": {
      "path": "/usr/bin/cc-runtime"
    },
    "custom": {
      "path": "/usr/local/bin/my-runc-replacement",
      "runtimeArgs": [
        "--debug"
      ]
    }
  },
  "default-address-pools":[
    {"base":"172.80.0.0/16","size":24},
    {"base":"172.90.0.0/16","size":24}
  ]
}
```



### 热更新

```bash
kill -SIGHUP $(pidof dockerd)
```



## Docker开启Api

### 修改服务文件

```bash
vim /usr/lib/systemd/system/docker.service
```

在 ExecStart=/usr/bin/dockerd 后面直接添加 -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock （注意端口8088自己随便定义，别跟当前的冲突即可）

### 修改配置daemon

```bash
vim /etc/docker/daemon.json
添加
"insecure-registries":["填你的harbor服务器地址"]
```

### 重启Docker

```bash
systemctl daemon-reload && systemctl restart docker
```

### 查看状态

```bash
docker -H [senhao.top]:2375 info
```

### 查看端口监听

```bash
netstat -anp|grep 2375
```

注意：本人测试时Docker服务会自动修改端口为2275，暂不知原因，感兴趣的大佬可以去排查下原因

## Docker日志

参考文章：[https://www.cnblogs.com/spec-dog/p/12624470.html](https://www.cnblogs.com/spec-dog/p/12624470.html)

### Docker日志

在启动容器的时候添加参数

```shell
--log-driver local  --log-opt max-size=10m  --log-opt max-file=3
```

例：

```shell
docker run -d --name nginx -p 80:80 --log-driver local  --log-opt max-size=10m  --log-opt max-file=3  --restart=always nginx
```

此时你的容器日志会被以文件形式存于目录

`/var/lib/docker/containers/{container_id}/local-logs/container.log`

### 查看

进入想查看的容器文件夹

```shell
# tail -f container.log 
stdout倚ªº㯦L1:C 26 Jan 202s2 05:32:12.112 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo` 
stdout訜ªº㯦m1:C 26 Jan 2022 05:32:12.112 # Redis version=6.2.6, bits=64, commit=00000000, modified=0, pid=1, just started·
```

## Docker命令

### 查看容器

``` bash
docker ps -as
```

### 启动容器

``` bash
docker start 容器id
```

### 删除所有容器

``` bash
docker rm $(docker ps -aq)
```

### 删除所有镜像

``` bash
docker rmi -f $(docker images -qa)
```

### 查看容器信息

``` bash
docker inspect [容器名称]
```

### 查看容器日志

``` bash
docker logs --tail="10" <CONTAINER ID>
```

### 设置Docker容器开机自启

``` bash
docker update –restart=always <CONTAINER ID>
```

### Docker容器重启

``` bash
docker restart [容器ID或容器名]
```

### 重启Docker

``` bash
systemctl restart docker
```

### 关闭容器的开机自启

``` bash
docker update --restart=no <CONTAINER ID>
```

### 查看网络模式

``` bash
docker network ls
```

### 查看容器占用资源

```bash
# 实时查看状态
docker stats
# 可以通过--no-stream来查看当前状态[推荐]
docker stats --no-stream
```

### 容器启动不退出

```bash
docker run -d nginx tail -f /dev/null
#或者
docker run -d nginx /bin/bash ping 127.0.0.1
#或者
docker run -d nginx /bin/bash -c "while true;do echo hello docker;sleep 1;done"

#使用交互式启动，使主进程不退出
docker run -i [CONTAINER_NAME or CONTAINER_ID]
```

## DockerFile

### 构建后端项目

> dockerfile

``` bash
#base image
FROM java:8
#home dir
ENV HOME /file-server
#jar name
ENV SERVER base-web-1.0-SNAPSHOT.jar
#server port
ENV PORT 8080
#console code
ENV LANG C.UTF-8
#time zone
ENV TZ=Asia/Shanghai
#create dir
RUN mkdir ${HOME}
#time configuration
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
#copy .jar file
ADD ${SERVER} ${HOME}/${SERVER}
#work path
WORKDIR ${HOME}
#expose server port
EXPOSE ${PORT}
#server run cmmand
ENTRYPOINT ["java","-jar","${SERVER}"]
```

> install.sh

``` bash
#!/bin/bash
docker stop senhao_file
docker rm senhao_file
docker rmi fileserver:v1
docker build -t fileserver:v1 .
docker run -d -p 9000:9000 -v /home/docker/file-server/:/file-server/file/ --name senhao_file fileserver:v1
```

> log.sh

``` bash
#!/bin/bash
docker logs -f -t senhao_file
```

### Nginx SSL

> dockerfile

``` dockerfile
FROM nginx:stable

# copy the custom website into the image
COPY index.html /usr/share/nginx/html/

# copy the SSL configuration file into the image
COPY ssl.conf /etc/nginx/conf.d/ssl.conf

# download the SSL key and certificate into the image
COPY 2_senhao.top.key /etc/nginx/ssl/nginx.key
COPY 1_senhao.top_bundle.crt /etc/nginx/ssl/nginx.crt

# expose the https port
EXPOSE 443
```

> ssl.conf

``` bash
cat << '__EOF' > ssl.conf
server {
listen       443 ssl;
server_name  localhost;

ssl_certificate /etc/nginx/ssl/nginx.crt;
ssl_certificate_key /etc/nginx/ssl/nginx.key;

location / {
root   /usr/share/nginx/html;
index  index.html index.htm;
}
}
__EOF
```

> install.sh

``` bash
#!/bin/bash
docker stop senhao_nginx
docker rm senhao_nginx
docker rmi nginx:v1
docker build -t nginx:v1 .
docker run --name senhao_nginx -d -p 443:443 -v /home/docker/nginx/log:/var/log/nginx -v /home/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /home/docker/nginx/conf.d:/etc/nginx/conf.d -v /home/docker/nginx/html:/usr/share/nginx/html nginx:v1
```

> log.sh

``` bash
#!/bin/bash
docker logs -f -t senhao_nginx
```

## Docker逆向

```bash
#构建
docker build --rm -t pegleg/whaler .
#配置别名
alias whaler="docker run -t --rm -v /var/run/docker.sock:/var/run/docker.sock:ro pegleg/whaler"
#使用
whaler -sV=1.36 nginx:latest
```

## Docker安装MySQL

### Docker 中下载 MySQL

```bash
docker pull mysql:latest
```

### 数据卷挂载启动

```bash
docker run -d -p 3306:3306 \
-v /data/docker/mysql/my.cnf:/etc/mysql/conf.d/mysqld.cnf \
-v /data/docker/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=root \
--restart always \
--name mysql \
mysql:latest
```

### 普通启动

```bash
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
```

### 进入容器

```bash
docker exec -it mysql bash
```

## Docker安装PostgreSQL

### 下载镜像

```bash
docker pull postgres:11
```

### 启动

```bash
docker run --name postgres \
-v /data/docker/postgres/data:/var/lib/postgresql/data \
-e POSTGRES_PASSWORD=postgres \
--restart always \
-p 5432:5432 \
-d postgres:11
```

## Docker安装Mongo

### 下载镜像

```bash
docker pull mongo
```

### 启动镜像(无安全验证)

```bash
docker run -d --restart=always \
-p 27017:27017 \
--restart always \
--name mongo \
-v /home/docker/mongo/data/db:/data/db \
mongo
```

### 启动镜像(安全验证)

```bash
docker run -d --restart=always \
-p 27017:27017 \
-v /data/docker/mongo/data/db:/data/db \
--name mongo \
--restart always \
-d mongo --auth
```

### 创建账户

> 进入容器

```bash
docker exec -it mongo mongo admin
```

> 创建管理员账户

```bash
db.createUser({ user:'admin',pwd:'123456',roles:[ { role:'userAdminAnyDatabase', db: 'admin'},"readWriteAnyDatabase"]});
```

### 安装mongo-express

```bash
docker pull mongoexpress
```

### 启动Mongo-express

```bash
docker run --link dark_mongo:mongo -p 8081:8081 mongo-express
```

## Docker安装Redis

### 下载镜像

```bash
docker pull redis:latest
```

### 数据卷挂载启动

### 创建文件

```bash
mkdir -p /home/docker/redis/{data,conf}
```

> 写入文件

```bash
tee /home/docker/redis/conf/redis.conf <<-'EOF'
#允许远程连接
bind *
#安全模式
protected-mode no
#持久化
appendonly yes
#密码 
requirepass 123456
EOF
```

### 启动

```bash
docker run --name redis \
-p 6379:6379 \
-v /data/docker/redis/data:/data \
-v /data/docker/redis/conf/redis.conf:/etc/redis/redis.conf \
--restart always \
-d redis redis-server /etc/redis/redis.conf
```

## docker安装Nginx

### 下载镜像

```bash
#拉取镜像
docker pull nginx:latest
#启动
docker run --name senhao_hexo -d -p 443:443 -p 80:80 senhao_hexo:latest
```

### Dockerfile

```dockerfile
FROM nginx:stable

# copy the custom website into the image
#COPY index.html /usr/share/nginx/html/
COPY dist /usr/share/nginx/html/

# copy the SSL configuration file into the image
COPY nginx.conf /etc/nginx/conf.d/nginx.conf

# download the SSL key and certificate into the image
COPY example.com.key /etc/nginx/ssl/nginx.key
COPY example.com_bundle.crt /etc/nginx/ssl/nginx.crt

# expose the https port
EXPOSE 443
```

#### install.sh

```sh
docker stop nginx
docker rm nginx
docker rmi nginx:latest
docker build -t nginx:latest .
docker run --name nginx -d -p 443:443 -p 80:80 nginx:latest
```

#### ssl.conf

```nginx
server {
listen       443 ssl;
server_name  example.com;

ssl_certificate /etc/nginx/ssl/nginx.crt;
ssl_certificate_key /etc/nginx/ssl/nginx.key;
ssl_session_timeout 5m;
ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;

#       location / {
#               root   /usr/share/nginx/html;
#               index  index.html index.htm;
#       }
        location / {
                root /usr/share/nginx/html/hexo;
                index  index.html index.htm;
        }
}

#http转https
server {
    listen 80;
    server_name example.com;
    rewrite ^(.*)$ https://${server_name}$1 permanent;
}

server {
    listen 80;
    server_name www.example.com;
    rewrite ^(.*)$ https://${server_name}$1 permanent;
}
```

## Docker安装Nacos

### 下载镜像

```bash
docker pull nacos/nacos-server
```

### 创建文件

```bash
mkdir -p /data/docker/nacos/{init.d,logs}
```

> 配置文件/home/docker/nacos/init.d/custom.properties内容如下

```bash
tee /data/docker/nacos/init.d/custom.properties <<-'EOF'
management.endpoints.web.exposure.include=*
EOF
```

### 启动

```bash
docker run -d -p 8848:8848 -e MODE=standalone \
-v /data/docker/nacos/init.d/custom.properties:/home/nacos/init.d/custom.properties \
-v /data/docker/nacos/logs:/home/nacos/logs \
--restart always \
--name nacos nacos/nacos-server
```

## Docker安装GitLab

### 下载镜像

```bash
docker pull gitlab/gitlab-ce
```

### 启动

```bash
docker run -d  -p 443:443 -p 80:80 -p 222:22 \
--name gitlab --restart always \
-v /home/docker/gitlab/config:/etc/gitlab \
-v /home/docker/gitlab/logs:/var/log/gitlab \
-v /home/docker/gitlab/data:/var/opt/gitlab \
gitlab/gitlab-ce
```

### 配置

```bash
# gitlab.rb文件内容默认全是注释
vim /home/gitlab/config/gitlab.rb
# 配置http协议所使用的访问地址,不加端口号默认为80
external_url 'http://[服务器IP]'
# 配置ssh协议所使用的访问地址和端口
gitlab_rails['gitlab_ssh_host'] = '[服务器IP]'
gitlab_rails['gitlab_shell_ssh_port'] = 222 # 此端口是run时22端口映射的222端口
#配置时区
gitlab_rails['time_zone'] = 'Asia/Shanghai'
```

### 重启gitlab容器

```bash
docker restart senhao_gitlab
```

### 重置密码

```bash
#进入容器
docker exec -it senhao_gitlab /bin/bash
#打开控制台
gitlab-rails console -e production
#id为1的用户
user=User.where(id:1).first
#输入密码
user.password='xusenhao'
#确认密码
user.password_confirmation='xusenhao'
#保存
user.save!
```

### 配置邮件服务

```bash
### Email Settings
gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = '你的QQ邮箱'
gitlab_rails['gitlab_email_display_name'] = '陈洋'
gitlab_rails['gitlab_email_reply_to'] = '你的QQ邮箱'
gitlab_rails['gitlab_email_subject_suffix'] = ''


### GitLab email server settings
###! Docs: https://docs.gitlab.com/omnibus/settings/smtp.html
###! **Use smtp instead of sendmail/postfix.**

gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.qq.com"
gitlab_rails['smtp_port'] = 465 
gitlab_rails['smtp_user_name'] = "你的QQ邮箱"
gitlab_rails['smtp_password'] = "授权码，不是邮箱密码" 
gitlab_rails['smtp_domain'] = "smtp.qq.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true

#重启服务
docker restart senhao_gitlab
#测试邮件发送
root@192:/# gitlab-rails console
irb(main):002:0> Notify.test_email('961363629@qq.com', 'Message Subject', 'Message Body').deliver_now

#注意：
#测试邮件时报错501
Net::SMTPSyntaxError: 501 mail from address must be same as authorization user

原因：配置文件gitlab.rb中### Email Settings没有设置邮箱地址，意思就是发送邮件的地址和SMTP认证的账号必须一致
解决办法：配置好### Email Settings即可

#测试邮件时报错SSLError
原因：此类报错通常是SMTP服务器没有使用SSL，但是配置却开启了
解决办法：注释掉以下配置
# gitlab_rails['smtp_enable_starttls_auto'] = true
# gitlab_rails['smtp_tls'] = true
```

## Docker安装Harbor

> 依赖docker-compose

### 下载

```bash
https://github.com/goharbor/harbor/releases/download/v2.4.1/harbor-online-installer-v2.4.1.tgz
```

### 解压

```bash
tar -zxvf harbor-online-installer-v2.3.1.tgz -C /usr/local/
```

### 修改文件

```bash
cp harbor.yml.tmpl harbor.yml
vim harbor.yml
```

修改hostname参数为本机IP或域名

### 安装

```bash
./install.sh
```

### 启动和重启

```bash
# 停止Harbor
docker-compose down -v

# 重启Harbor
docker-compose up -d
```

### Docker配置

```bash
#编辑文件
vim /etc/docker/daemon.json
#添加
"insecure-registries": ["[ip]:[port]"]
#重启
systemctl daemon-reload
systemctl restart docker
```

## Docker安装Jenkins

### 下载镜像

```bash
docker pull jenkins/jenkins
```

### 创建文件夹

```bash
mkdir -p /home/docker/jenkins_mount
```

#### 授权

```bash
chmod 777 /home/docker/jenkins_mount
```

### 启动

```bash
docker run -d -p 10240:8080 -p 10241:50000 \
-v /data/docker/jenkins_mount:/var/jenkins_home \
-v /etc/localtime:/etc/localtime \
--restart always \
--name jenkins jenkins/jenkins
```

### 访问

访问ip:10240 可以修改映射端口

## Docker安装RabbitMQ

### 启动

```bash
docker run -d --name rabbitmq \
-p 5672:5672 -p 15672:15672  \
--restart always \
--hostname myRabbit \
-e RABBITMQ_DEFAULT_VHOST=my_vhost  \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin \
rabbitmq:management
```

## Docker安装Kafka

```bash
docker run -d --restart=always --name kafka \
-p 9092:9092 \
-e KAFKA_BROKER_ID=0 \
-e KAFKA_ZOOKEEPER_CONNECT=10.1.52.149:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://10.1.52.149:9092 \
-e KAFKA_LISTENERS=PLAINTEXT://:9092 \
wurstmeister/kafka
```

### Kafka测试

#### 单机

使用docker启动Kafka.

```bash
#进入到/opt/kafka/bin目录
cd /opt/kafka/bin
#1. 创建topic
./kafka-topics.sh --create --zookeeper $KAFKA_ZOOKEEPER_CONNECT --replication-factor 1 --partitions 1 --topic test-topic
#2. 查看 topic
./kafka-topics.sh --list --zookeeper $KAFKA_ZOOKEEPER_CONNECT
#3. 运行一个生产者
./kafka-console-producer.sh --broker-list localhost:9092 --topic test-topic
#4. 运行一个消费者
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic --from-beginning
```

## Docker安装rancher/server

### 启动

```bash
docker run -di --name=rancher -p 9090:8080 rancher/server
```

## Docker安装zookeeper

### 下载镜像

``` bash
docker pull zookeeper
```

### 单机部署

``` bash
docker run -d -p 2181:2181 \
-v /data/docker/zoo/data:/data \
-v /data/docker/zoo/datalog:/datalog \
-v /data/docker/zoo/logs:/logs \
--name zookeeper \
--restart always zookeeper
```

## Docker安装Spark

> 依赖docker-compose

### pull image

``` bash
docker pull singularities/spark
```

### 创建docker-compose

``` yaml
version: "2"
 
services:
  master:
    image: singularities/spark
    command: start-spark master
    hostname: master
    ports:
      - "6066:6066"
      - "7070:7070"
      - "8080:8080"
      - "50070:50070"
  worker:
    image: singularities/spark
    command: start-spark worker master
    environment:
      SPARK_WORKER_CORES: 1
      SPARK_WORKER_MEMORY: 2g
    links:
      - master
```

### 启动

``` bash
docker-compose up -d
```

### 查看运行状态

``` bash
docker-compose ps
```

### 停止

``` bash
docker-compose stop
```

### 删除

``` bash
docker-compose rm
```

## Docker部署Minio

```bash
docker run \
--name minio \
-d --restart=always \
-p 9000:9000 -p 9001:9001 \
-e "MINIO_ROOT_USER=minio" \
-e "MINIO_ROOT_PASSWORD=password" \
-v /data/minio/data:/data \
-v /data/minio/config:/root/.minio minio/minio server /data \
--console-address ":9001"
```

### Nginx反代Minio

nginx反代后直接使用域名无法访问对象资源(访问资源对象端口为9000)，需要添加Buckets代理

```nginx
server {
listen       443 ssl;
server_name  domain.com;

ssl_certificate /etc/nginx/ssl/nginx.crt;
ssl_certificate_key /etc/nginx/ssl/nginx.key;
ssl_session_timeout 5m;
ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;
client_max_body_size 0;

        location / {
                proxy_pass http://domain.com:9001;
                proxy_set_header Host $http_host;
                proxy_http_version      1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";

        }

}
server {
    listen 80;
    server_name domain.com;
    rewrite ^(.*)$ https://${server_name}$1 permanent;
}
```

## Docker 搭建一个短链服务

```shell
docker run -itd --name shorter \
--restart always \
-e DB_DRIVE=redis \
-e REDIS_DRIVE=single \
-e REDIS_HOSTS=127.0.0.1:6379 \
-e REDIS_PASSWORD=123456 \
-e REDIS_DB=1 \
-e SHORT_URI=example.com \
-e LOG_PATH=/var/log/ \
-e LOG_LEVEL=debug \
-p 8080:8080 \
dudulu/shorter:latest
```

## Docker拷贝

### 语法

``` bash
docker cp 容器进程ID:文件/文件夹路径  主机目的路径
```

### 例

``` bash
docker cp 017149848c20:/home/ /home/user/
```

## Docker打包与提交

### 容器

#### 语法

``` bash
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

#### 例

``` bash
docker commit -a "dark" -m "mysql" a568eff33275 mysql-dark:1.0
```

OPTIONS说明：-a :提交的镜像作者； -c :使用Dockerfile指令来创建镜像； -m :提交时的说明文字；-p :在commit时，将容器暂停。此时，docker images 就会出现打包好的镜像

``` bash
docker export 98ca36> ubuntu.tar
cat ubuntu.tar | sudo docker import - ubuntu:import
```

### 镜像

``` bash
docker save ubuntu:load>/root/ubuntu.tar
docker load<ubuntu.tar
```

### 批量处理

``` bash
#打包全部镜像
docker save $(docker images | grep -v REPOSITORY | awk 'BEGIN{OFS=":";ORS=" "}{print $1,$2}') -o harborAll.tar
#加载所有镜像
docker load -i harborAll.tar
```

## Docker 清理

### 容器清理

```bash
#删除停止运行的容器
docker container prune
#删除所有容器(包括停止的、正在运行的)
docker rm -f $(docker ps -aq)
```

### 镜像清理

 ```bash
 #可以列出所有悬挂状态的镜像
 docker image ls -f dangling=true
 #删除悬挂状态的镜像
 docker image rm $(docker image ls -f dangling=true -q)
 #清除悬挂镜像
 docker image prune
 #删除所有镜像。但正在被容器使用的镜像无法删除。
 docker image rm $(docker image ls -q) 
 #清除none版本镜像
 docker rmi $(docker images | grep "none" | awk '{print $3}')
 ```

### 数据卷清理

```bash
#删除不再使用的数据卷
docker volume prune
#删除不再使用的数据卷
docker volume rm $(docker volume ls -q) 
```

### 缓存清理

```bash
#删除 build cache
docker builder prune
```

### 一键清理

```bash
注意，这两个命令会把你暂时关闭的容器，以及暂时没有用到的 Docker 镜像都删掉了
#查看 Docker 的磁盘使用情况
docker system df
#清理磁盘，删除关闭的容器、无用的数据卷和网络，以及 none 镜像
docker system prune
#清理得更加彻底，可以将没有容器使用 Docker镜像都删掉
docker system prune -a
```

## Docker迁移

```bash
#查看默认存储路径
docker info |grep  "Docker Root Dir"
#停止docker服务
systemctl stop docker
#新建文件夹
mkdir /home/docker
#迁移文件
rsync -r -avz /var/lib/docker  /home/docker/
#配置docker服务
vi /usr/lib/systemd/system/docker.service

[Service]
ExecStart=/usr/bin/dockerd  --graph=/home1/docker/lib/docker
#重启服务
systemctl daemon-reload && systemctl restart docker
```

## Docker Proxy

参考地址：[Docker Proxy](https://blog.csdn.net/qq_39698985/article/details/123748820)

在执行`docker pull`时，是由守护进程`dockerd`来执行。因此，代理需要配在dockerd的环境中。而这个环境，则是受systemd所管控，因此实际是systemd的配置。

### Image

```bash
#创建文件
mkdir -p /etc/systemd/system/docker.service.d
touch /etc/systemd/system/docker.service.d/proxy.conf
#配置proxy.conf
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:8080/"
Environment="HTTPS_PROXY=http://proxy.example.com:8080/"
Environment="NO_PROXY=localhost,127.0.0.1,.example.com"
#重启
systemctl daemon-reload && systemctl restart docker
```

### Container

在容器运行阶段，如果需要代理上网，则需要配置 `~/.docker/config.json`。以下配置，只在Docker 17.07及以上版本生效。

```bash
{
 "proxies":
 {
   "default":
   {
     "httpProxy": "http://proxy.example.com:8080",
     "httpsProxy": "http://proxy.example.com:8080",
     "noProxy": "localhost,127.0.0.1,.example.com"
   }
 }
}
```

这个是用户级的配置，除了 proxies，docker login 等相关信息也会在其中。而且还可以配置信息展示的格式、插件参数等。

此外，容器的网络代理，也可以直接在其运行时通过 -e 注入 http_proxy 等环境变量。这两种方法分别适合不同场景。config.json 非常方便，默认在所有配置修改后启动的容器生效，适合个人开发环境。在CI/CD的自动构建环境、或者实际上线运行的环境中，这种方法就不太合适，用 -e 注入这种显式配置会更好，减轻对构建、部署环境的依赖。当然，在这些环境中，最好用良好的设计避免配置代理上网。

### Docker Build

虽然 `docker build` 的本质，也是启动一个容器，但是环境会略有不同，用户级配置无效。在构建时，需要注入 http_proxy 等参数。

```bash
docker build . \
    --build-arg "HTTP_PROXY=http://proxy.example.com:8080/" \
    --build-arg "HTTPS_PROXY=http://proxy.example.com:8080/" \
    --build-arg "NO_PROXY=localhost,127.0.0.1,.example.com" \
    -t your/image:tag
```

注意：无论是 docker run 还是 docker build，默认是网络隔绝的。如果代理使用的是 localhost:3128 这类，则会无效。这类仅限本地的代理，必须加上 --network host 才能正常使用。而一般则需要配置代理的外部IP，而且代理本身要开启 Gateway 模式。
重启生效

代理配置完成后，reboot 重启当然可以生效，但不重启也行。

docker build 代理是在执行前设置的，所以修改后，下次执行立即生效。Container 代理的修改也是立即生效的，但是只针对以后启动的 Container，对已经启动的 Container 无效。

dockerd 代理的修改比较特殊，它实际上是改 systemd 的配置，因此需要重载 systemd 并重启 dockerd 才能生效。