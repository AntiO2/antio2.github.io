---
title: "查询计划练习"
description: CMU15-445 2022Fall Homework
date: 2023-03-23T20:08:01+08:00
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
links:
  - website: https://15445.courses.cs.cmu.edu/fall2022/files/hw3-clean.pdf
    title: Query Execution Exercise
---

## 0x1 Sorting Algorithms

> We have a database file with fourteen million pages (N = 14,000,000 pages), and we want to sort it using external merge sort. Assume that the DBMS is not using double buffering or blocked I/O, and that it uses quicksort for in-memory sorting. Let B denote the number of buffers.

有14,000,000页，B表示Buffer池容量

**(a) Assume that the DBMS has eight buffers. How many passes does the DBMS need to perform in order to sort the file**

假如Buffer池容量为8块，在归并排序中，七块用于输入，一块用于输出结果📕。**第一次归并**：每8个为一组，读入8个页，因为内存可以容纳这8个页的所有元组，所以直接在内存中进行排序。这样就形成了（14,000,000/8）个归并段。**第二阶段**：接下来，将归并段每七个为一组，再次排序输出到第八个内存块中（满了就输出，一共会输出7*7次(因为有7个归并段，每个归并段有7页)）。所以，一共会进行$$log_7(14,000,000/8)+1$$ 趟归并。计算结果为**9**

**(b)Again, assuming that the DBMS has eight buffers. What is the total I/O cost to sort the file**

每页每趟归并都要经过一次读入和一次读出，故结果为$$14,000,000\times 2\times 9 = 252,000,000$$

**(c) What is the smallest number of buffers B that the DBMS can sort the target file using only eight passes**

按照归并排序的思想，第一次内存中排序形成**14,000,000/B**个归并段，然后在新的一趟中，每七个归并段一起输出成一个新的归并段。也就是要求解$$log_{B-1}(14,000,000/B)+1<8$$，也即求最小的B使得$$B*(B-1)^7>14,000,000$$,解得**B=9**。

**(d)Suppose the DBMS has forty-two buffers. What is the largest database file (expressed in terms of N, the number of pages) that can be sorted with external merge sort using four passes**

💬如果内存池有42页，那么第一次归并每组为42页，接下来3趟每次处理41个归并段，答案为$$42*41^3=2,894,682$$

## 0x2 Join Algorithms

Consider relations R(a, b, c), S(a, d), and T(a, e, f) to be joined on the common attribute a. Assume that there are no indexes available on the tables to speed up the join algorithms. 

- There are B = 445 pages in the buffer
- Table R spans M = 1,500 pages with 80 tuples per page 
- Table S spans N = 4,500 pages with 150 tuples per page 
- Table T spans O = 200 pages with 250 tuples per page 

❓Answer the following questions on computing the I/O costs for the joins. You can assume the simplest cost model where pages are read and written one at a time. You can also assume that you will need **one** buffer block to hold the evolving output block and **one** input block to hold the current input block of the inner relation. You may ignore the cost of the writing of the final results.

计算以下情况的IO消耗：

**(a)Block nested loop join with S as the outer relation and R as the inner relation:**

```
for 将(B-2)块外关系表读入外关系S入内存: (执行4500/(B-2)次)
	for 读入外关系 (执行1500次)：
		进行连接操作，输出到预留块中
```

S为外表，读入消耗4500次IO，然后循环扫描内表R。扫描时，内存中的（B-2）块外表和一块内表连接，读完1500次外表后，执行下一次循环 ，再读入（B-2）块外表，然后再读1500页内表进行连接。所以一共消耗$$4500+1500*ceil(4500/443)=21000$$

**(b)Block nested loop join with R as the outer relation and S as the inner relation:**

🧩和a思路相同，内外表互换，$$1500+4500*ceil(1500/443)=19500$$

**(c)Sort-merge join with S as the outer relation and R as the inner relation:**

sort-merge是先按照键进行排序，然后按照顺序进行连接。

- What is the cost of sorting the tuples in R on attribute a

排序R要多少次IO?使用第一题归并排序的思路，要进行$$1+log_{443}(1500/445)=2$$趟，然后读写为$$2\times 2\times 1500=6000$$次。

- What is the cost of sorting the tuples in S on attribute a?

$$1+log_{443}(4500/445)=2$$,然后读写$$2\times 2\times 4500=18000$$次。

- What is the cost of the merge phase in the worst-case scenario?

⚠️worst-case scenario指的是所有排序的值都一样，这种情况下要进行MxN次页比较。$$1, 500 \times  4, 500 = 6, 750, 000$$

- What is the cost of the merge phase assuming there are no duplicates in the join attribute?

M+N次，想象有两个游标，如果没有重复值的话，两个游标会一直移动。$$1, 500 + 4, 500 = 6, 000$$

- Now consider joining R, S and then joining the result with T. Suppose the cost of the final merge phase is 800 and assume that there are no duplicates in the join attribute. How many pages did the join result of R and S span?

这道题是倒推，已知R和S的连表和T最后join的cost为800，因为T有200页，故R和S最后生成600页（用没有重复值情况下的M+N的公式）。

**(d)Hash join with S as the outer relation and R as the inner relation. You may ignore recursive partitioning and partially filled blocks.**

S作为外表,R为内表，忽略一个bucket满了的情况

- What is the cost of the probe phase?

（M+N）,把内存中相应的bucket取出来，然后用h2再创建哈希索引进行连接操作。这里用哈希函数2的原因是：需要生成不同的哈希值，因为之前分组时用h1后，在一个bucket中的都是相同的哈希值，再用h1已经没意义了，不能快速找到属性真正相同的元组。🌲

- What is the cost of the partition phase?

2*(M+N),因为要扫描一遍R和S,把元组分到不同的bucket中，然后再写回内存。📊

**(e)Assume that the tables do not fit in main memory and that a high cardinality of distinct values hash to the same bucket using your hash function h1. Which of the following approaches works the best?** 

有大量不同的值都被哈希到了同样的bucket中， 应该如何处理？

- ✅Create hashtables for the inner and outer relation using h1 and rehash into an embedded hash table using h2 != h1 for large buckets （用h2生成不同的哈希值，因为不同的哈希函数再出现这种碰撞概率是很小的）
- Create hashtables for the inner and outer relation using h1 and rehash into an embedded hash table using h1 for large buckets 
- Use linear probing for collisions and page in and out parts of the hashtable needed at a given time 
- Create 2 hashtables half the size of the original one, run the same hash join algorithm on the tables, and then merge the hashtables together

## 0x3 Query Execution

**(a) Which processing model has on average the smallest working buffer per operator invocation? Ignore optimizations like projection pushdown. Select only one answer.** 

- Iterator ✅
- Materialization 
- Vectorization

Iterator ,又被称为流水线模型，可以一个元组一个元组处理。Materialization 是把所有元组合并，然后推给父节点。Vectorization Model有点像两个模型的折中，一次拿多个tuple，既不拿完，也不一个一个拿。所以使用缓存最小的是Iterator，因为大部分情况它可以处理单个元组。

**(b) In the iterator processing model, the logic of an operator is independent of its children and parents. (i.e., the code does not case on what type of iterator the children or parents are)** ✅

判断题。流水线模型中操作和父子结点无关。这是对的，因为一个operator实现next、close、emit这三个接口就行了。

**(c) In the vectorized processing model, each operator that receives input from multiple children *requires* multi-threaded execution to generate the Next() output tuples from each child** ❌

向量模型一定要多线程？是可以单线程的

**(d) The iterator processing model often leads to good code locality (in the instruction cache sense).**

locality：近邻。流水线模型一定能带来好的代码局部性？不一定，因为如果是一个tuple一个tuple处理的话，假如这个流水线很长，比如通过函数f1、f2、f3.....f10000,每处理一个tuple,就会将代码从内存读到缓存中，可能将f10000读到cache中时，前面的函数已经被踢出去了，所以代码局部性真的不行。而且不同函数之间的next可能是远程调用（它们在内存不在同一个页上）❌

**(e) An index scan is always better (fewer I/O operations, faster run-time) than a sequential scan, regardless of the processing model.** ❌

索引扫描一定比顺序扫描好？不一定，如果是非聚簇索引，扫描会不停地读取不同的页。而且通过优化顺序扫描，通过把下一页提前拿到内存中，可能还更快。Postgre的优化器如果预测到要拿超过10%的行，那么会进行顺序扫描。

## 0x4 总结

通过这次作业，加深了对归并排序，连接操作，查询执行的理解。这些知识点在课件上全都有，上课也讲过。通过定量分析也是对操作的流程有了掌握。