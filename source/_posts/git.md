---
title: Git笔记
date: 2022-04-10 13:18:34
tags: git
categories: git
description: git笔记.
---
## Git配置

### Git配置

> 设置Git的Name与Eamil

``` bash
git config --global user.name "senhao-xu"
```

``` bash
git config --global user.email "xusenhao1123@163.com"
```

> 生产SSH

``` bash
ssh-keygen -t rsa -C "senhao-xu"
```

### Git回滚

> Git强制回滚

``` bash
#获取需要恢复的版本号
git reset --hard [需要恢复的版本号]
#强制推送到
git push -f -u origin [分支]
```

### Git 剔除 add 的文件

> 清除缓存

``` bash
git rm --cached “文件路径”
```

> 物理删除

``` bash
git rm --f “文件路径”
```

### Git 代码合并

```bash
#切换到当前分支
git checkout [current]
#拉取当前分支代码
git pull
#切换到主分支
git checkout master
#把分支的代码merge到主分支
git merge [current]
#推送代码,完成合并
git push
```





