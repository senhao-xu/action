---
title: Java脚手架archetype
date: 2022-04-16 13:23:06
tags: BackEnd
categories: BackEnd
description: Java搭建企业级脚手架
---
## Java使用archetype搭建搭建企业级脚手架

### 前言

Archetype原型 也就是说为项目生成一个原型，可以把这个项目发布，其他人就可以直接通过命令构建一个原型项目了。

### 搭建

首先你需要有自己搭建的一个maven项目，添加你项目中经常使用的一些common，或者你需要的文件

在该的项目中的pom.xml文件中添加一下信息

```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-archetype-plugin</artifactId>
                <version>3.0.1</version>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

进入该项目所在路径的终端使用`mvn archetype:create-from-project`进行构建，构建完成后便会生成一个archetype项目模板。此时你就完成了构建

### 使用

使用`mvn install`把该模板安装到maven，打开你的idea添加maven模板，此刻你便完成了所有操作。

### 发布到maven私服
参考该教程[地址](https://blog.csdn.net/wangshuminjava/article/details/120721369),发布完成。