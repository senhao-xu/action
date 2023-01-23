---
title: sql笔记
date: 2022-04-10 13:18:34
tags: SQL
categories: SQL
description: sql笔记.
---

## MYSQL

### 创建用户

% : 一般为IP地址

``` sql
CREATE USER '用户名称'@'%' IDENTIFIED BY '密码';
```

### 赋予权限

``` sql
GRANT SELECT, INSERT, UPDATE, REFERENCES, DELETE, CREATE, DROP, ALTER, INDEX, CREATE VIEW, SHOW VIEW ON `数据库名称`.* TO '用户名'@'IP';
```

### 查重

``` sql
-- 单字段（name）
-- 查出所有有重复记录的所有记录
select * from user where `user`.`name` in
     (select `user`.`name` from user group by name having count(`user`.`name`)>1);
```

``` sql
-- 多字段（nick_name,password）
-- 查出所有有重复记录的记录
select * from user where (nick_name,password) in
     (select nick_name,password from user group by nick_name,password where having count(nick_name)>1);
```

### 去重

``` sql
-- 只可以查到单字段
select distinct `user`.`name` from `user`
```

### 修改密码

> 8版本

``` sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的密码'
```

### MySQL重置自增ID

``` sql
alter table 表名 drop id;

alter table 表名 add id bigint primary key not null auto_increment first;
```

## PostgreSQL

### id重新自增

``` sql
TRUNCATE TABLE 表名 RESTART IDENTITY CASCADE;
```

### 表id自增

```sql
CREATE SEQUENCE IF NOT EXISTS [t_table_id_seq];
```

### 强制删除现有连接

```sql
SELECT pg_terminate_backend(pg_stat_activity.pid) FROM pg_stat_activity WHERE datname = '[database]' AND pid <> pg_backend_pid();
```

### 删除数据库

```sql
drop database [database];
```

## Cypher

> 查询

``` CQL
MATCH (n) RETURN n LIMIT 25
```

> 删除图

```CQL
MATCH (n) DETACH DELETE n
```

> 创建

```CQL
--可以有多个lable
CREATE (<node-name>:<label-name>)
```

> SET

```CQL
--给现有的lable添加属性
match (n:Person) set n.name = 'ceshi' return n
```

> DELETE 会删除标签本身

```CQL
--删除节点或关系的属性 | 删除节点及相关节点和关系
DELETE <node-name-list>
--删除name为ceshi的数据   
MATCH (n:Dept) WHERE n.name = "ceshi" DELETE n
```

> REMOVE 不会删除标签本身

```CQL
--删除节点或关系的标签 | 删除节点或关系的属性
--类似于删除表中的列
REMOVE <property-name-list>
--删除name为ceshi的属性 
MATCH (person { name:"ceshi" }) REMOVE person.name RETURN person
```
