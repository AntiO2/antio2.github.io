---
title: 使用pg_stat_statements进行tpch性能分析
description: 
date: 2024-10-13T16:17:37+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: true
tags:
  - Database
  - PolarDB
  - 性能
  - 存储
categories:
  - 数据库
---

## 安装与配置


1. 首先参考[pg\_stat\_statement如何安装部署使用-CSDN博客](https://blog.csdn.net/weixin_37692493/article/details/109281169) 安装插件。
2. 安装完成后，需要连接数据库手动进行以下操作：
	- 执行`CREATE EXTENSION pg_stat_statements;`

在配置中写入如下语句。
 
```shell	
pg_stat_statements.max = 100 
pg_stat_statements.track = all 
pg_stat_statements.track_utility = true
pg_stat_statements.enable_superuser_track = true
```

通过下面的语句查询当前的配置

```sql
SELECT name, setting, unit, category, short_desc
FROM pg_settings
WHERE name LIKE 'pg_stat_statements.%';
```


## 使用

查询视图：

```sql
SELECT * FROM pg_stat_statements;
```

重置视图：

```sql
SELECT pg_stat_statements_reset();
```

