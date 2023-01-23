---
title: Npm笔记
date: 2022-04-10 13:18:34
tags: FrontEnd
categories: FrontEnd
description: Npm笔记.
---
## NPM笔记

#### 下载node.js

> 官网网站 ： [https://nodejs.org/zh-cn/](https://nodejs.org/zh-cn/)

#### 安装淘宝镜像(cnpm)

```bash
npm install -g cnpm --registry=http://registry.npm.taobao.org
```

> 配置npm永久使用(换源)

```bash
npm config set registry https://registry.npm.taobao.org
```

#### 查看源

> ```
> npm get registry
> ```

#### 修改回默认源

> ```
> npm config set registry https://registry.npmjs.org/
> ```

#### 安装webpack

```bash
npm install webpack -g 
```

#### 安装vue

#### 默认版本

```bash
npm install vue-cli -g
```

#### 3.\*版本

```bash
npm install @vue/cli -g
```

#### npm install chromedriver问题

> npm install报错：chromedriver@2.27.2 install: node install.js

1、如果执行过npm install，先删除 node\_modules 文件夹，不然运行的时候可能会报错 2、执行下面的命令

```bash
npm install chromedriver --chromedriver_cdnurl=http://cdn.npm.taobao.org/dist/chromedriver
```

3、再执行 npm install 即可正常下载

> 分析

经分析发现，某些版本下，chromedriver 的 zip 文件 url 的响应是 302 跳转，而在 install.js 里使用的是 Node.js 内置的 http 对象的 get 方法无法处理 302 跳转的情况；而在另外一些情况下，则是因为 googleapis.com 被墙了，此时即使采用科学 上网的方法也仍然无法获取文件。

#### node-sass

镜像地址改为淘宝镜像

> npm config set sass\_binary\_site=[https://npm.taobao.org/mirrors/node-sass](https://npm.taobao.org/mirrors/node-sass)

#### Vue 发布

> 使用npm run build 打包 会得到一个dist文件夹

> nginx配置

```bash
#location / {
        #    root  html/dark/dist/;
        #    index index.html; # 主页
        #}
```
