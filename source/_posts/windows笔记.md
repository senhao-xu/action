---
title: windows笔记
date: 2022-09-10 12:58:05
tags: Windows
categories: Windows
description: windows笔记.
---

## 激活及配置

### 查看当前Windows激活信息

> slmgr.vbs -dli

> slmgr.vbs -dlv

### 查看Windows永久激活

> slmgr.vbs -xpr

### Windows密钥注册表地址

> 计算机\HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SoftwareProtectionPlatform

### Windows11右击菜单恢复到Windows10

> ·运行“regedit”，开启注册表编辑器，定位到“HKEY\_CURRENT\_USER\SOFTWARE\CLASSES\CLSID”；
>
> 　　·接着，右键点击“CLSID”键值，新建一个名为{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}的项；
>
> 　　·右键点击新创建的项，新建一个名为InprocServer32的项，按下回车键保存；
>
> 　　·最后选择新创建的项，然后双击右侧窗格中的默认条目，什么内容都不需要输入，按下回车键。
>
> 　　保存注册表后，重启explorer.exe，即可看到右键菜单恢复成旧样式了。
>
> 　　如果想要恢复成为Win11的设计，那么删掉InprocServer32的项就可以了。

### 联想Edge由你的组织管理

> HKEY\_LOCAL\_MACHINE\SOFTWARE\Policies\Microsoft\Edge
>
> 把其中的SmartScreenEnabled和SmartScreenPuaEnabled两项删除后将edge浏览器关闭重新打开

### Windows卸载密钥

> slmgr.vbs /upk

### 安装新密钥

> slmgr /ipk \[新密钥]

### 查看office相关信息

> 进入office目录
>
> cd "C:\Program Files\Microsoft Office\Office16"
>
> 查看已安装的密钥：
>
> cscript ospp.vbs /dstatus
>
> 卸载密钥：（将命令中的XXXXX改为上一步查到的后五位字符）
>
> cscript "C:\Program Files\Microsoft Office\Office16\ospp.vbs" /unpkey:XXXXX

### 桌面菜单注册表

> 计算机\HKEY\_CLASSES\_ROOT\Directory\Background\shell

### 桌面菜单修改到右边

```bash
shell:::{80F3F1D5-FECA-45F3-BC32-752C152E456E}
```

### powershell cmdlet、函数、脚本文件或可运行程序的名称错误处理

```bash
Set-ExecutionPolicy Unrestricted
```

### 追踪ip节点

``` bash
tracert -d \[ip]
```

### Windows 运行程序图标丢失解决

``` bash
#使用cmd命令窗执行一下命令
taskkill /im explorer.exe /f

cd /d %userprofile%\appdata\local

del iconcache.db /a

start explorer.exe

exit
```

### 取消右击AMD Radeon Software

```bash
#注册表
HKEY_CLASSES_ROOT\Directory\Background\shellex\ContextMenuHandlers
#删除ACE文件夹
```

## 脚本

### git提交脚本

把脚本放在需要提交的git仓库下

``` bash
@echo off
 
title GIT一键提交
color 3
echo 当前目录是：%cd%
echo;
 
echo 开始添加变更：git add .
git add .
echo;
 
set /p declation=输入提交的commit信息:
git commit -m "%declation%"
echo;
 
echo 将变更情况提交到远程自己分支：git push origin master
git push origin master
echo;
 
echo 执行完毕！
echo;
 
pause
```

### zookeeper启动脚本

把路径修改为自己zk的地址

``` bash
title zookeeper
f:
cd F:\TechnicalPackage\apache-zookeeper-3.5.9-bin\bin
zkServer.cmd

pause
```

## 查看端口占用

### 查找端口

``` bash
netstat -aon|findstr 8080
```

### 查找pid进程

``` bash
tasklist|findstr 2524
```

### 结束进程

``` bash
kill 2524
```

## Hyper-V

### 导入

```bash
Import-VM -Path 'F:\backup\centos\Virtual Machines\F5AC5B5C-76A4-429F-AD24-379FD660DDC7.vmcx' -Copy -GenerateNewId -VhdDestinationPath 'C:\Hyper-V\centos' -VirtualMachinePath 'C:\Hyper-V\centos'
```

## 安装mysql

### 下载

> [https://dev.mysql.com/downloads/mysql/](https://dev.mysql.com/downloads/mysql/)

在MySQL目录下创建my.ini文件

> 适用8版本

``` bash
[mysqld]
# 设置3306端口
port=3306
# 设置mysql的安装目录
basedir=F:\TechnicalPackage\mysql-8.0.25-winx64
# 设置mysql数据库的数据的存放目录
datadir=F:\TechnicalPackage\mysql-8.0.25-winx64\Data
# 允许最大连接数
max_connections=200
# 允许连接失败的次数。
max_connect_errors=10
# 服务端使用的字符集默认为utf8mb4
character-set-server=utf8mb4
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 默认使用“mysql_native_password”插件认证
#mysql_native_password
default_authentication_plugin=mysql_native_password
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8mb4
#开启binlog
server_id=1918
log_bin = mysql-bin
binlog_format = ROW
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8mb4
```

> 适用5版本

``` bash
[mysqld]
##skip-grant-tables=1
port = 3306
basedir=E:\TechnicalPackage\mysql-5.7.30-winx64
datadir=E:\TechnicalPackage\mysql-5.7.30-winx64\data
max_connections=200
character-set-server=utf8
default-storage-engine=INNODB
explicit_defaults_for_timestamp=1

[mysql]
default-character-set=utf8
```

### 初始化

第一次安装时会有默认密码，安装完成后修改密码

``` bash
mysqld --initialize --console
```

### 安装服务

``` bash
mysqld --install
```

### 启动服务

``` bash
net start mysql
```

## 安装PostGreSQL

### 下载

> [https://www.enterprisedb.com/download-postgresql-binaries](https://www.enterprisedb.com/download-postgresql-binaries)

### 初始化

F:\TechnicalPackage\pgsql\data 为你postgresql的路径

``` bash
initdb.exe -U postgres -A password -E utf8 -W -D F:\TechnicalPackage\pgsql\data
```

### 启动(如注册Windows服务这一步不用运行)

``` bash
pg_ctl -D ^"F^:^\TechnicalPackage^\pgsql^\data^" -l logfile start
```

### 注册服务

``` bash
pg_ctl register -N PostgreSQL -D "F:/TechnicalPackage/pgsql/data"
```

### 启动服务

``` bash
net start PostgreSQL 
```

## 安装Redis

### 下载redis

> [https://github.com/tporadowski/redis/releases](https://github.com/tporadowski/redis/releases)

### 安装服务

``` bash
redis-server.exe --service-install redis.windows.conf --loglevel verbose
```

### 取消安装

``` bash
redis-server --service-uninstall
```

## 安装MongoDB

### 下载

**注：推荐使用msi文件安装，二进制安装会出现问题**

> 下载地址： [https://www.mongodb.com/download-center#communit](https://www.mongodb.com/download-center#community)

## 安装Neo4J

> **需要Java环境**

### 下载

> [https://neo4j.com/download-center/#community](https://neo4j.com/download-center/#community)****

### **通过控制台启动Neo4j程序**&#x20;

``` bash
neo4j.bat console
```

### **安装为Windows服务**

``` bash
#安装
neo4j install-service
#卸载
neo4j uninstall-service
```

### **其他命令**

``` bash
#启动
neo4j start
#停止
neo4j stop
#重启
neo4j restart
#查看状态
neo4j status
```

### **neo4j.conf**

``` bash
#Configure the Neo4j Browser to time out logged in users after this idle period. Setting this to 0 indicates no limit.
#Valid units are: 'ns', 'μs', 'ms', 's', 'm', 'h' and 'd'; default unit is 's'
#添加会话过期
browser.credential_timeout=1000
```
