---
title: "查询计划与优化"
description: 
date: 2023-03-21T20:31:24+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
tags:
  - CMU15-445
  - Database
categories:
  - 数据库
---

## 逻辑计划与物理计划

逻辑计划与物理计划两者的区别在于：逻辑计划告诉你算子的控制流程，而物理计划指定算子的具体执行方法。

具体来说，一个逻辑计划的算子为Scan, 物理计划可以生成Index Scan或者Seq Scan。再比如Join: 可能存在Sort Join,Hash Join...

### 优化搜索中止

因为查找一个最优的查询计划是一个NP问题，所以我们需要找到停下来的方法。

1. 如果优化器运行一定时间，中止。
2. 如果一个计划优化得足够号，中止。
3. 当穷举完毕，中止。

## Heuristic-Based

启发式算法：通过规则，执行从逻辑到物理的转化方法。

**对于符合要求的，总是应用规则。所以这是静态的优化方式。**

缺点在于**各种配置需要写在代码里面**。



## Heuristic + Cost-Based Join Search

**Example: System R**

使用静态的规则进行最初的优化。然后使用动态规划进行join reorder(通过分治)。

 

最初只进行左深树的优化。

**分层搜索**

Starburst:

- Query Rewrite
- Plan Optimization

**火山优化**

基于成本的优化器。

 查询优化概述

**查询优化的两种方法：**

- Rules:通过静态的条件判断，来重写查询。通过查看catalog而不是数据。
- Cost-based 枚举SQL的所有方案，并且预估成本然后选择成本低的。

### 查询优化的流水线

1. SQL Rewriter 对SQL语句进行重写标记上额外信息（可选）
2. Parser 进行编译，将sQL转化为语法树
3. Binder 将SQL引用的符号转化为内部标记。比如`select * from foo`,会通过字符串"foo"来寻找相应的表。
4. Logical Plan
5. Tree Rewiter通过查询system catalog来进行优化
6. 进入Optimizer,计算成本模型
7. 生成物理计划

**逻辑计划和物理计划的区别：**

逻辑计划生成关系代数表达式，物理计划使用具体的操作符，是实际执行的底层逻辑。逻辑计划和物理计划不一定一一对应。



## 优化方法

**Predicate Pushdown,通过提前执行谓语（从语法树上往下推），来减少工作量**

- Selections：越早执行过滤操作越好。
- Projections: 提早执行，来最小化要执行的元组数据。

**单一关系模型**

- 顺序查询
- 聚簇索引
- 索引扫描

首先判断是否sargable(Search Argument  Able)是否又对应索引。

现在大多数DBMS采用heuristics(启发式)而不是精确的模型。

一个sargable的query能很容易被启发式的模型执行。

**多个关系模型**

可以枚举的：

1. 操作顺序
2. 操作的方法：比如hash、sort-merge、nested loop...
3. 拿到数据的方法：Index#1、Index#2、Seq Scan...

随着join数量的增加，可行的所有方法也会增加，故不能使用枚举，要限制搜索的数量来在可接受的时间内找到优化的方案。

- 自底向上：从nothing逐渐构建方案。例子：IBM System R, DB2, MySQL, Postgres, most open-source DBMSs
- 自顶向下：从想要的结果开始，然后从语法树上到下优化。例子：MSSQL, Greenplum, CockroachDB, Volcano
- 遗传算法 PostgreSQL(GEQ)

**自底向上优化：System R**

## 研究

[Home - Database of Databases (dbdb.io)](https://dbdb.io/)