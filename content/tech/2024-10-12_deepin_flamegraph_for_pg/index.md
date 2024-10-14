---
title: 使用性能分析工具优化PostgreSQL
description: Linux 性能分析工具使用，以及针对PolarDB的性能分析示例
date: 2024-10-12T23:56:54+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: true
tags:
  - Database
  - Linux
  - PolarDB
  - 存储
  - 性能
categories:
  - 数据库
  - 笔记
---
之前我在[火焰图介绍](/tech/intro_flame_graph) 中写了大概如何安装perf工具以及进行简单的火焰图实验。最近因为对PolarDB需要进行调优，所以记录一下如何针对 [GitHub - ApsaraDB/PolarDB-for-PostgreSQL: A cloud-native database based on PostgreSQL developed by Alibaba Cloud.](https://github.com/ApsaraDB/PolarDB-for-PostgreSQL) 进行性能分析。

https://lawrencezx.github.io/blogs/2022-3-Linux-Dynamic-Tracing.html