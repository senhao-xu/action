---
title: network-agent
date: 2022-04-10 13:18:34
tags: Proxy
categories: Proxy
description: network-agent笔记.
---

## VPN

### 搭建X-UI

```bash
docker run -itd --network=host \
    -v $PWD/db/:/etc/x-ui/ \
    -v $PWD/cert/:/root/cert/ \
    --name x-ui --restart=unless-stopped \
    enwaiax/x-ui:latest
```

### 安装BBR

```bash
wget "https://github.com/chiakge/Linux-NetSpeed/raw/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh
```

### 搭建v2board

#### docker搭建

docker搭建:   [GitHub](https://github.com/GZ1903/docker_v2board)

```bash
docker run -d \
--name=v2board \
--privileged=true \
--restart always \
-v /root/v2board/nginx:/usr/share/nginx/html/v2board \
-v /root/v2board/data:/var/lib/mysql \
-p 80:80 \
-p 443:443 \
gz1903/v2board:1.6.0
```

| MySQL_Default_USER | ROOT                                    |
| ------------------ | --------------------------------------- |
| MySQL_Default_PASS | [v2board@qq.com](mailto:v2board@qq.com) |

| V2Board_Admin_USER | [v2board@qq.com](mailto:v2board@qq.com) |
| ------------------ | --------------------------------------- |
| V2Board_Admin_PASS | [v2board@qq.com](mailto:v2board@qq.com) |

##### 版本升级

```shell
#Interact
docker exec -it v2board /bin/bash
#Get the new version of v2board
git clone https://github.com/v2board/v2board.git /usr/share/nginx/html/v2board/
#Install
cd /usr/share/nginx/html/v2board && sh /usr/share/nginx/html/v2board/init.sh
#mysqlpass => v2board@qq.com 
#Give v2board permission
chmod -R 777 /usr/share/nginx/html/v2board
#Restart queue
supervisorctl restart v2board
```

#### DockerCompose搭建

##### 使用 Docker 安装 V2Board

##### 安装 Docker

```
curl -fsSL https://get.docker.com | bash
curl -L "https://github.com/docker/compose/releases/download/1.25.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod a+x /usr/local/bin/docker-compose
# 创建个软链接，以后用 dc 命令来代替 docker-compose
rm -rf which dc  # 若系统中存在 dc 则删除，这个 dc 就是个计算器，完全没有用
ln -s /usr/local/bin/docker-compose /usr/bin/dc
```

##### 克隆代码

```
git clone https://github.com/v2board/v2board-docker.git
cd v2board-docker/
git submodule update --init
echo '  branch = dev' >> .gitmodules
git submodule update --remote
```

修改'caddy.conf'
第一行'http://你的域名 {'

##### 启动环境

```
dc up -d
```

##### 安装配置 V2Board

```
dc exec www bash
bash-5.0# wget https://getcomposer.org/download/2.5.1/composer.phar
bash-5.0# php composer.phar install
bash-5.0# php artisan v2board:install
数据库地址： mysql
数据库名：v2board
数据库用户名：root
数据库密码：v2boardisbest
```

#### 手动

官方搭建手册：[ V2Board 使用手册](https://docs.v2board.com/deploy/aapanel.html)

```bash
git clone https://github.com/GZ1903/docker_v2board.git /usr/local/src/docker_v2board && cd /usr/local/src/docker_v2board && chmod +x docker_v2board.sh && ./docker_v2board.sh
```

#### 后端对接

参考： [使用docker安装 · GitBook (xrayr-project.github.io)](https://xrayr-project.github.io/XrayR-doc/xrayr-xia-zai-he-an-zhuang/install/docker.html)

### 搭建Telegram代理

```bash
docker run --name s5 -d --restart=always -p 1111:1111 -e "USERS=cloud:112233" egregors/socks5-server
```

### 部署warp

参考地址：[https://p3terx.com/archives/cloudflare-warp-configuration-script.html](https://p3terx.com/archives/cloudflare-warp-configuration-script.html)

```bash
bash <(curl -fsSL git.io/warp.sh) d
```

### 虚拟软路由

#### [Hyper-V搭建openWrt](https://blog.yoitsu.moe/tech\_misc/hyper\_v\_with\_openwrt.html)

**准备工作 - Hyper-V 管理器、映像和创建虚拟机**

要使用 Hyper-V 的话，汝得有一个 Windows 8.1/10 专业版或者企业版， 或者 Windows Server 2012 以后的 Windows Server 才行。同时也要求汝的 CPU 支持必要的虚拟化技术。 （Intel VT-x 或者 AMD-V 什么的）可以通过 PowerShell 里的 systeminfo 命令确定是否可以开启 Hyper-V 。

在满足了前置要求之后，可以这样启用 Hyper-V：

```bash
# 如果提示找不到命令的话，换以管理员身份运行的 PowerShell 窗口。
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

或者用“启用或关闭 Windows 功能”也是可以的。

**下载和转换 OpenWrt 映像**

接下来到 OpenWrt 的网站上下载映像 （这时候最新的版本是 19.07，于是链接就是 [https://downloads.openwrt.org/releases/19.07.7/targets/x86/64/](https://downloads.openwrt.org/releases/19.07.7/targets/x86/64/) ）

这里咱们选择包含有内核和 rootfs 的 combined 那俩， 区别在于 squashfs 那个安装好以后系统分区像在普通路由器一样是只读的，可以实现像是升级或者复位等功能。 不过咱大概会用检查点来实现这些，那就直接用 ext4 的好了。下载和解压以后汝大概就可以得到一个像是 openwrt-19.07.7-x86-64-combined-ext4.img 这样的文件了。

但是 Hyper-V 只能用微软自己的 VHD 或者 VHDX 格式，所以还需要转换一下。 转换的方法有很多，咱这里给出咱的一个例子。

创建一个虚拟磁盘。这里用了 diskpart 命令。如果汝更偏好用磁盘管理操作的话， 可以参考 [这篇文章](https://docs.microsoft.com/zh-cn/windows-server/storage/disk-management/manage-virtual-hard-disks)

```bash
Microsoft DiskPart version 10.0.21332.1000
Copyright (C) Microsoft Corporation.
On computer: DESKTOP-H6MANBV

# 创建一个虚拟磁盘文件
# create vdisk file=<汝要存放虚拟磁盘文件的位置> maximum=<它的最大大小，以 MiB 为单位> type=expandable
# 假如汝有充足的硬盘空间的话，可以把上面的 type=expandable 改为 type=fixed 创建一个固定大小的虚拟磁盘。
# 可以提高一些虚拟磁盘的性能。当然动态扩展对 OpenWrt 来说也够用就是了……
DISKPART> create vdisk file=c:\test.vhd maximum=8192 type=expandable
    100 percent completed
DiskPart successfully created the virtual disk file.

# 选择刚才创建的虚拟磁盘
# select vdisk file=<汝要存放虚拟磁盘文件的位置>
DISKPART> select vdisk file=c:\test.vhd
DiskPart successfully selected the virtual disk file.

# 挂载刚才选择的虚拟磁盘。
DISKPART> attach vdisk
    100 percent completed
DiskPart successfully attached the virtual disk file.

# 初始化虚拟磁盘为 MBR 分区表。
# OpenWrt 官方编译的版本不支持 UEFI 启动，所以就用 MBR 了。
DISKPART> convert mbr
DiskPart successfully converted the selected disk to MBR format.

# 在下一步操作完成之后卸载虚拟磁盘
DISKPART> detach vdisk
DiskPart successfully detached the virtual disk file.
```

接下来把这个映像写入虚拟磁盘就 OK 啦。（以及咱发现 Rufus 能把虚拟磁盘识别出来）

### OpenWrt

个人使用的编译版本地址

> [https://netflixcn.com/miji/46.html](https://netflixcn.com/miji/46.html)

密码：netflixcn.com

### 编译openwrt

使用github编译openwrt,可以参考该[教程](https://p3terx.com/archives/build-openwrt-with-github-actions.html)比较全面.
