---
title: hexo搭建教程
date: 2022-04-10 15:18:34
tags: blog
categories: blog
description: hexo搭建教程.
---

## hexo搭建教程

基于Windows系统进行搭建

### 前置条件

1. nodejs
2. git

### 安装hexo

我们采用`Hexo`来创建我们的博客网站，`Hexo` 是一个基于`NodeJS`的静态博客网站生成器，使用`Hexo`不需开发，只要进行一些必要的配置即可生成一个个性化的博客网站，非常方便。点击进入 [官网](https://hexo.io/zh-cn/)。

#### 安装hexo

```bash
npm install -g hexo-cli
```

#### 查看版本

```bash
hexo -v
```

#### 创建一个hexo

```bash
hexo init blog
```

##### 初始化

```bash
npm install
```

#### 本地启动

```bash
#生成静态文件 
hexo generate | hexo g
#进行预览
hexo server | hexo s
```

此时汝的一个简单的hexo-blog就搭建完成啦!

### 更换主题

`Hexo` 默认的主题不太好看，不过官方提供了数百种主题供用户选择，可以根据个人喜好更换，官网主题点 [这里](https://hexo.io/themes/) 查看。这里介绍两个主题的使用方法，`Next` 和 `Fluid`，个人比较喜欢`Fluid`。

#### 下载Fluid主题

[这里](https://github.com/fluid-dev/hexo-theme-fluid)是官方地址,该REAEMD.md中有较为详细的教程，这里不再重复.

```bash
git clone https://github.com/fluid-dev/hexo-theme-fluid.git ./themes
```

#### 创建一个post

```bash
hexo new [post]
```

此时在blog/source/_posts/，会出现创建的post。

### 发布

#### 私人服务器

1、打包汝的blog

```bash
hexo g
```

将`public` 目录下的文件复制到 `Linux` 服务器，使用nginx进行代理即可。

### 使用时出现的问题

#### 主页摘要

编写完成文章后，主页预览发现摘要显示部分内容。解决方案有两种：

1. 使用`<!--more-->`进行截断
2. 添加`description`信息

#### 标签和分类

##### 创建一个标签模块

```bash
hexo new page tags
```

此时你的`source`文件夹下有了`categories\index.md`，打开`index.md`文件将 title 设置为`title: 分类`。文章归入分类只需在文章的顶部标题下方添加`categories`字段，即可自动创建分类名并加入对应的分类中。

##### 创建一个分类模块

```bash
hexo new page categories
```

此时你的`source`文件夹下有了`tags\index.md`，打开`index.md`文件将 title 设置为`title: 标签` 。文章添加标签只需在文章的顶部标题下方添加`tags`字段，即可自动创建标签名并归入对应的标签中 。

