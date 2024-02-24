---
title: "学习Mysql 常用指令"
description: 
date: 2021-11-21T20:58:49+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
tags:
  - 笔记
categories:
  - 数据库
  - 笔记
---

## Sql数据类型

| 说明   | 数据类型           |
| ------ | ------------------ |
| 整形   | bit,int            |
| 小数   | decimal            |
| 字符串 | varchar,char       |
| 时间   | date,time,datetime |
| 枚举   | enum               |

## Mysql字段约束

| 约束参数       | 说明     |
| -------------- | -------- |
| primary_key    | 主键约束 |
| not null       | 非空约束 |
| unique         | 唯一约束 |
| default        | 默认约束 |
| AUTO_INCREMENT | 自增     |

## Mysql数据库服务端启动

|          | sudo service mysql [] |
| -------- | --------------------- |
| 查看状态 | status                |
| 启动     | start                 |
| 停止     | stop                  |
| 重启     | restart               |

## 配置文件地址

`/etc/mysql/mysql.conf.d`

# Mysql终端指令操作

## 登录客户端操作

1. 连接指令  

`mysql -u用户  -p密码`

1. 显示时间

`select now();`

3. 退出连接

`quit/ exit/ ctrl + d`

4. 创建用户

`CREATE USER anti@localhost IDENTIFIED BY '123456';`

## 数据库操作

| 说明                 | 指令                                  |
| -------------------- | ------------------------------------- |
| 查看所有数据库       | show databases                        |
| 查看当前所用的数据库 | select database()                     |
| 切换到指定数据库     | use [数据库名]                        |
| 创建数据库           | create database 数据库名 charset=utf8 |
| 删除数据库           | drop database[数据库名]               |

##  表操作

| 说明             | 指令                                                         |
| ---------------- | ------------------------------------------------------------ |
| 查看所有表       | show tables                                                  |
| 创建表           | create table 表名（字段名称 数据类型 可选约束 主键[不为空] 自增） |
| 改变表的字段类型 | alter table [表名] modify [field]                            |
| 删除表           | drop table [表名]                                            |

## Mysql-CRUD操作

### 查询数据

| 说明       | 指令                         |
| ---------- | ---------------------------- |
| 查询所有列 | select * from 表名           |
| 指定列查询 | select 列名(,列名) from 表名 |

## 插入数据

| 说明           | 备注                       | 指令                                      |
| -------------- | -------------------------- | ----------------------------------------- |
| 全列插入       | 值的插入顺序和列的顺序一致 | insert into 表名 values(...);             |
| 部分列插入     | 值的顺序和给出列的顺序对应 | insert into 表名（列1...） values(值...); |
| 全列多行插入   |                            | insert into 表名 values(...),(...),(...); |
| 部分列多行插入 |                            | ...                                       |

## 修改数据

```sql
update 表名 set 列1 = 值1，列2 = 值2... where 条件
例如
update students set age = 18，gender = '女'... where id = 6;
```

## 删除数据

```sql
delete from 表名 where 主键（例如 id = 66）
```

## 数据库备份导出

`mysqldump -u用户名 -p 数据库名字 表名字 > data.sql`

## 恢复导入

在`use 数据库后` `source data.sql`