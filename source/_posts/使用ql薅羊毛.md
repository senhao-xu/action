---
title: 使用青龙薅羊毛搭建教程
date: 2022-04-21 13:18:34
tags: docker
categories: docker
description: 搭建青龙获取京豆，薅羊毛.
---

## 安装

### 基于docker进行安装

```bash
docker run -dit \
-v $PWD/ql/config:/ql/config \
-v $PWD/ql/log:/ql/log \
-v $PWD/ql/db:/ql/db \
-p 5700:5700 \
--name qinglong \
--hostname qinglong \
--restart always \
whyour/qinglong:latest
```

执行命令后，在浏览器访问`ip:5700`

## 初始化环境

### 进入容器

```bash
docker exec -it qinglong bash
```

### 安装依赖命令

```bash
curl -fsSL https://ghproxy.com/https://raw.githubusercontent.com/shufflewzc/QLDependency/main/Shell/QLOneKeyDependency.sh | sh
```

### 添加依赖

在页面点击 `依赖管理`的右上角`添加依赖`. 对以下分类进行添加

#### nodeJs

```bash
crypto-js
prettytable
dotenv
jsdom
date-fns
tough-cookie
tslib
ws@7.4.3
ts-md5
jsdom -g
jieba
fs
form-data
json5
global-agent
png-js
@types/node
require
typescript
js-base64
axios
```

#### Python3

```bash
requests
canvas  
ping3
jieba
aiohttp
```

#### Linux

```bash
bizCode
bizMsg  
lxml
```

## 使用

### 添加定时任务

```bash
# 定时 秒 分 时 天 月 周
*/4 * * * *
# 命令
ql repo https://ghproxy.com/https://github.com/shufflewzc/faker2.git "jd_|jx_|gua_|jddj_|getJDCookie" "activity|backUp" "^jd[^_]|USER|utils|ZooFaker_Necklace.js|JDJRValidator_Pure|sign_graphics_validate" "main"

ql repo https://ghproxy.com/https://github.com/KingRan/KR.git "jd_|jx_|jdCookie" "activity|backUp" "^jd[^_]|USER|utils|function|sign|sendNotify|ql|JDJR"
```

### 添加环境变量

chrome browser go to ` https://m.jd.com` get cookies: `xx_key` and `xx_pin` , go to the dashboard add envirment `JD_COOKIE`, value is `xx_key=xxx;xx_pin=xxx`

## 参考

https://github.com/zhangguanzhang/docker-compose/tree/master/qinglong

https://www.dujin.org/18884.html





