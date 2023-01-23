---
title: go
date: 2022-05-07 15:44:06
tags: BackEnd
categories: BackEnd
description: go笔记.
---

## GoLang

### 打包

#### Linux

```bash
set CGO_ENABLED=0
set GOOS=linux
set GOARCH=amd64
go build -o main-linux main.go
```

#### Windows

```bash
set CGO_ENABLED=0
set GOOS=windows
set GOARCH=amd64
go build -o main-windows.exe main.go
```

#### Mac

```bash
set CGO_ENABLED=0
set GOOS=darwin
set GOARCH=amd64
go build -o  main-mac main.go
```

#### 打包多系统

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o abc-demo-linux main.go
CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build -o abc-demo-mac main.go
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build -o abc-demo-windows.exe main.go
```

#### 参考地址

[传送门](https://blog.csdn.net/k393393/article/details/122674509)

### 配置国内代理

```bash
go env -w GOPROXY=https://goproxy.cn,direct
```



