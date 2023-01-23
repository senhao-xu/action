---
title: Linux笔记
date: 2022-09-10 13:18:34
tags: Linux
categories: Linux
description: Linux笔记.
---

## Linux

## 更改主机名称

``` bash
hostnamectl set-hostname [new name]
```

## 更换源为阿里云

``` bash
#备份
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
#下载阿里源
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
#清除缓存、生成缓存
yum clean all     # 清除系统所有的yum缓存
yum makecache     # 生成yum缓存
#查看源
yum repolist all     #查看所有的yum源
yum repolist enabled #查看可用的yum源 
```

## 更换为官方源

```bash
#备份源
cd /etc/yum.repos.d/
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup 
#重建官方源
rpm -Uvh --force http://mirror.centos.org/centos-7/7.9.2009/os/x86_64/Packages/centos-release-7-9.2009.0.el7.centos.x86_64.rpm
#重建缓存
yum clean all
yum makecache
```

## 网络配置

``` bash
#进入网络文件夹
cd /etc/sysconfig/network-scripts/ 
#找到类似于 ifcfg-eth0 的文件， 也可能是别的
vi ifcfg-eth0
#将以下值修改为yes
ONBOOT=yes
#更改为静态IP
#修改ifcfg-eth0   下列信息获取自己的网络属性
BOOTPROTO=static         # 使用静态IP地址，默认为dhcp
IPADDR=192.168.31.1   	   # 设置的静态IP地址
NETMASK=255.255.255.0    	# 子网掩码
GATEWAY=192.168.31.1  		 # 网关地址
DNS1=192.168.31.1     		 # DNS服务器
#重启服务
service network restart
```

## 内核升级

```bash
#查看当前内核版本
uname -r
#更新源
yum -y update
#导入ELRepo仓库的公共密钥
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
#安装ELRepo仓库的yum源
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
#查询可用内核版本
yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
#安装最新的稳定版本内核
yum -y --enablerepo=elrepo-kernel install kernel-lt
#查看系统上的所有可用内核
sudo awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
#设置 grub2
grub2-set-default 0
#重启
reboot
```



## 时间同步

```bash
#下载ntpdate包
yum install -y ntpdate
#同步Windows时间
ntpdaet time.windows.com
```

## 安装yum wget ...指令

``` bash
yum install -y wget
yum install -y vim
```

## 安装net-tools包

``` bash
#查找ifconfig在那个包下
yum search ifconfig 
#安装依赖包
yum install net-tools.x86_64
```

## 端口

``` bash
#检查端口被哪个进程占用
netstat -lnp|grep 8090  

#查看进程的详细信息
ps 6832

#结束端口
kill -9 28937 

#查看监听的端口
netstat -lnpt 1234
```

## chmod

```bash
#批量给指定文件授权
#语法 
chmod [-R] [权限] `find [文件夹路径] -type f -regex ".*\.\([后缀]\|[后缀]\)"`
#例：表示给当前文件夹下所有sh文件授予777权限
chmod -R 777 `find ./ -type f -regex ".*\.\(sh\)"`
```

## Vim

```bash
#替换字符
%s/[s1]/[s2]/g
#显示行号
:set number | set nu
#批量修改指定后缀文件编码
#例子: 
for i in `find ./ -type f -regex ".*\.\(sh\)"`;do vi $i -c 'set ff=unix | wq';done
#语法:
for i in `find [path] -type f -regex ".*\.\([后缀]\)"`;do vi $i -c 'set ff=unix | wq';done
#vim自动添加行数
vim ~/.vimrc
set nu
```

## 防火墙

``` bash
 # 查看防火墙状态
firewall-cmd --state 

#防火墙开机自启
systemctl enable firewalld

#开启防火墙
systemctl start firewalld

#查看防火墙所有开放的端口
firewall-cmd --zone=public --list-ports 

# 开放3306端
firewall-cmd --zone=public --add-port=3306/tcp --permanent  

#关闭5672端
firewall-cmd --zone=public --remove-port=3306/tcp --permanent 

# 配置立即生效
firewall-cmd --reload  

#关闭防火墙
systemctl stop firewalld.service

#关闭防火墙（永久）
systemctl disable firewalld.service
```

## 开启服务

``` bash
#查看服务
ps aux | grep 端口

#启动docker
systemctl start docker

#关闭docker
systemctl stop docker

#查看docker的运行状态
systemctl status docker
```

## 存储

``` bash
# 查看当前文件夹使用大小
du -h
# 查看系统磁盘
df -hT
#查看当前目录下文件及目录大小[把*换掉，查具体的文件]
du -sh *
```

## 查看进程

> \-A 显示所有进程（同-e） -a 显示当前终端的所有进程 -u 显示进程的用户信息 -o 以用户自定义形式显示进程信息 -f 显示程序间的关系
>
> USER 进程所有者的用户名 PID 进程号 START 进程激活时间 %CPU 进程自最近一次刷新以来所占用的CPU时间和总时间的百分比 %MEM 进程使用内存的百分比 VSZ 进程使用的虚拟内存大小，以K为单位 RSS 驻留空间的大小。显示当前常驻内存的程序的K字节数。 TTY 进程相关的终端 STAT 进程状态，包括下面的状态： D 不可中断 Uninterruptible sleep (usually IO) R 正在运行，或在队列中的进程 S 处于休眠状态 T 停止或被追踪 Z 僵尸进程 W 进入内存交换（从内核2.6开始无效） X 死掉的进程 < 高优先级 N 低优先级 L 有些页被锁进内存 s 包含子进程 + 位于后台的进程组； l 多线程，克隆线程 TIME 进程使用的总CPU时间 COMMAND 被执行的命令行 NI 进程的优先级值，较小的数字意味着占用较少的CPU时间 PRI 进程优先级。 PPID 父进程ID WCHAN 进程等待的内核事件名

``` bash
# java(需要查看的进程名称)ps -ef|grep java  
```

## Java相关

启动jar

> nohup java -jar \[jar文件] >\[日志文件] &

查看实时日志

> tail -f \[日志文件]

## 使用publicKey登录

**生成sshKey**

> 使用xshell工具生成

**上传ssh**

``` bash
cd .ssh#上传ssh文件到该文件夹下
```

**导入ssh**

``` bash
cat [ssh文件名称] >> authorized_keys
```

**取消密码的登录**

``` bash
passwd -d root
```

### NC

> 语法： nc \[服务器地址] \[端口]

``` bash
例：nc www.baodu.com 80
```

> 查看连接状态： netstat -natp

> 发送请求

``` bash
语法 ：[请求方式] [URL] [协议]例：GET / HTTP/1.0--注需要换行，所以要敲两次回车
```

## RZ/SZ使用

> 安装

``` bash
yum install lrzsz-0.12.20-27.1.el6.x86_64.rpm -y
```

> rz 可以多文件

``` bash
#在终端直接输入rz即可 rz [-y] 覆盖文件
```

> sz 可以多文件

``` bash
#在终端输入sz [文件1] [文件2]
```

## 修改open files

> 需要root权限

> 设置open files

``` bash
echo ulimit -n 65535 >>/etc/profile
```

> 生效

``` bash
source /etc/profile
```

## 安装NFS

### 创建共享目录

```sh
mkdir -p /data/kubernetes
```

### 安装组件

```sh
yum -y install nfs-utils rpcbind
```

### 编辑配置文件

```sh
vi /etc/exports
/data/kubernetes 192.168.31.0/24(rw,sync,no_root_squash)
```

### 启动服务

```sh
service nfs start
service rpcbind start
```

### 设置开机自启动

```sh
systemctl enable nfs
systemctl enable rpcbind
```

### 测试

```sh
showmount -e 192.168.31.179
```

## yum离线下载

### yumdownloader的安装及使用

#### 安装

``` sh
yum install -y yum-utils 
#安装yum-plugin-downloadonly插件
yum install -y yum-plugin-downloadonly
```

#### 使用

> 语法

``` sh
yum install -y --downloadonly --downloaddir=[目录] [软件包]
```

> 例

``` sh
yum install -y --downloadonly --downloaddir=/root/mysql mysql
```

## 文件追加及覆盖

### 追加

如果你追加的文件没有换行，追加进去的文件也没有换行

```shell
echo "* soft nofile 65535 * hard nofile 65535" >> /etc/security/limits.conf
```

### 覆盖

如果之前的文件存在则会覆盖

```shell
tee /etc/security/limits.conf <<-'EOF'
* soft nofile 65535
* hard nofile 65535
EOF
```

## Linux安装Python

```bash
#获取tarbal
wget https://www.python.org/ftp/python/3.10.4/Python-3.10.4.tgz

#解压
tar -xvf Python-3.10.4.tgz

#安装openssl1.1.1，python3.10需要openssl1.1.1版本以上
yum install -y epel-release
yum install -y openssl11 openssl11-devel
ln -sf /usr/lib64/pkgconfig/openssl11.pc /usr/lib64/pkgconfig/openssl.pc

#安装依赖
yum install zlib-devel bzip2-devel openssl openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make readline libffi-devel -y
 
#编译安装
cd Python-3.10.4
./configure --prefix=/usr/local/python3.10.4
make
make install

#命令创建软连接
ln -s /usr/local/python3.10.4/bin/python3 /usr/bin/python3
ln -s /usr/local/python3.10.4/bin/pip3 /usr/bin/pip3

#版本查看
python3 -V
pip3 -V
```

## Linux安装GoLang

### 下载安装包

```bash
wget https://golang.google.cn/dl/go1.19.3.linux-amd64.tar.gz
```

### 解压

```bash
tar -zxvf go1.19.3.linux-amd64.tar.gz -C /usr/local/
```

### 配置环境变量

```bash
export GOROOT="/usr/local/go"
export GOPATH=$HOME/go
export GOBIN=$GOROOT/bin
export PATH=$PATH:$GOBIN
```

### 配置国内代理

```bash
go env -w GOPROXY=https://goproxy.cn,direct
```

### 验证

```go
go version
```

## 命令

### 获取IP （依赖ipconfig命令）

```bash
#获取有线网卡
echo eth0=`ifconfig  eth0 | head -n2 | grep inet | awk '{print$2}'`
#获取无线网卡
echo wlan0=`ifconfig  wlan0 | head -n2 | grep inet | awk '{print$2}'`
```

