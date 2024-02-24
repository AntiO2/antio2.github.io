---
title: "高频 SQL 50 题（基础版）"
description: 
date: 2023-11-30T01:32:28+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
tags:
  - LeetCode
  - Database
  - MySQL
categories:
  - 数据库
  - 笔记 
---



做了一下LeetCode的SQL50题题单, 记录一下有的没的的东西。

[高频 SQL 50 题（基础版） - 学习计划 ](https://leetcode.cn/studyplan/sql-free-50/) 



## 三值逻辑运算

在 [584. 寻找用户推荐人 - 力扣（LeetCode）](https://leetcode.cn/problems/find-customer-referee/description/?envType=study-plan-v2&envId=sql-free-50) 中。一个判断条件是要求某一列不是一个特定的值。

但是这一列可能是NULL。

第一次写了个错误方法：

```sql
SELECT name 
FROM customer
WHERE referee_id <> 2 
```

因为`WHERE` 过滤的条件为true，才会使用这条记录。但是 `null!=2` 的值不是`false`, 而是`unknown`。 所以上面的语句也不会选出列为NULL的记录。





三值逻辑运算的结果：

```sql
NOT unknown => unknown


OR:
true          OR unknown => true
unknown OR unknown => unknown
false         OR unknown => unknown

AND:
true          AND unknown => unknown
unknown AND unknown => unknown
false         AND unknown => false
```



## 字符串长度

[1683. 无效的推文 - 力扣（LeetCode）](https://leetcode.cn/problems/invalid-tweets/description/?envType=study-plan-v2&envId=sql-free-50)

- char_length(str)

1. 计算单位：字符
2. 不管汉字还是数字或者是字母都算是一个字符

- length(str)

1. 计算单位：字节
2. utf8编码：一个汉字三个字节，一个数字或字母一个字节。
3. gbk编码：一个汉字两个字节，一个数字或字母一个字节。

举个例子，假如一张表的一列内容为`十五字十五字十五字十五字十五字`

```sql
SELECT CHAR_LENGTH(content) FROM `tweets`; # 输出15
SELECT LENGTH(content) FROM `tweets` # 输出45
```

## HAVING,ON,WHERE 的区别

`WHERE`与`HAVING`的根本区别在于：

- `WHERE`子句在`GROUP BY`分组和聚合函数**之前**对数据行进行过滤；
- `HAVING`子句对`GROUP BY`分组和聚合函数**之后**的数据行进行过滤。



`ON`和`WHERE`:

- `ON` 可以作为`JOIN Conditions`
- `WHERE`在连接之后。

对于内连接没区别

但是对于外连接，比如表`T1(a,b)`,表`T2(a,c)`



`T1 LEFT JOIN T2 WHERE T1.a=1`和`T1 LEFT JOIN T2 ON(T1.a=1)`结果不同

通过JOIN算子是如何实现的，应该很好理解。

## 关于时间的函数

[SQL Date 函数 (w3school.com.cn)](https://www.w3school.com.cn/sql/sql_dates.asp)

- `NOW()      `             当前的日期和时间         

- `CURDATE()  `             当前日期                 

- `CURTIME() `          当前时间          

- `DATE(string)    `        提取给定时间字符串的日期

- `EXTRACT(unit FROM date)`  提取日期表达式的指定部分 

- `DATE_ADD(date,INTERVAL expr type)`

  - [550. 游戏玩法分析 IV - 力扣（LeetCode）](https://leetcode.cn/problems/game-play-analysis-iv/description/?envType=study-plan-v2&envId=sql-free-50)

  - expr是增加的单位量

  - | TYPE是时间间隔单位 |
    | ------------------ |
    | MICROSECOND        |
    | SECOND             |
    | MINUTE             |
    | HOUR               |
    | DAY                |
    | WEEK               |
    | MONTH              |
    | QUARTER            |
    | YEAR               |
    | SECOND_MICROSECOND |
    | MINUTE_MICROSECOND |
    | MINUTE_SECOND      |
    | HOUR_MICROSECOND   |
    | HOUR_SECOND        |
    | HOUR_MINUTE        |
    | DAY_MICROSECOND    |
    | DAY_SECOND         |
    | DAY_MINUTE         |
    | DAY_HOUR           |
    | YEAR_MONTH         |

  - ```sql
    SELECT OrderId,DATE_ADD(OrderDate,INTERVAL 2 DAY) AS OrderPayDate
    FROM Orders
    ```

- [DATE_SUB()](https://www.w3school.com.cn/sql/func_date_sub.asp)

- `DATEDIFF(date1,date2)`

  - 返回两个日期之间的天数
  - 若date1 > date2 返回正数。
  - 例如`DATEDIFF('2023-11-2','2023-11-1')=1`
  - [197. 上升的温度 - 力扣（LeetCode）](https://leetcode.cn/problems/rising-temperature/description/?envType=study-plan-v2&envId=sql-free-50)

- [MySQL DATE_FORMAT() 函数 (w3school.com.cn)](https://www.w3school.com.cn/sql/func_date_format.asp)

## 格式化输出

- [ROUND](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_round)

```mysql
mysql> SELECT ROUND(-1.23);
        -> -1
mysql> SELECT ROUND(-1.58);
        -> -2
mysql> SELECT ROUND(1.58);
        -> 2
mysql> SELECT ROUND(1.298, 1);
        -> 1.3
mysql> SELECT ROUND(1.298, 0);
        -> 1
mysql> SELECT ROUND(23.298, -1);
        -> 20
mysql> SELECT ROUND(.12345678901234567890123456789012345, 35);
        -> 0.123456789012345678901234567890
```

通过round舍入，第一个参数是需要四舍五入的数字。第二个是可选参数，表示保留的位数，默认为0。

- [truncate](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_truncate)

截断指定的小数位数，而不舍入。

## CROSS JOIN

[1280. 学生们参加各科测试的次数 - 力扣（LeetCode）](https://leetcode.cn/problems/students-and-examinations/description/?envType=study-plan-v2&envId=sql-free-50)

获得笛卡尔积。

## IF()、IFNULL()、NULLIF()、ISNULL()函数进行流程的控制

1、IF()函数的使用
IF(expr1,expr2,expr3)，如果expr1的值为true，则返回expr2的值，如果expr1的值为false，则返回expr3的值。

```
SELECT IF(TRUE,'A','B');    -- 输出结果：A
SELECT IF(FALSE,'A','B');   -- 输出结果：B
```


2、IFNULL()函数的使用
IFNULL(expr1,expr2)，如果expr1的值为null，则返回expr2的值，如果expr1的值不为null，则返回expr1的值。


```
SELECT IFNULL(NULL,'B');    -- 输出结果：B
SELECT IFNULL('HELLO','B'); -- 输出结果：HELLO
```

3、NULLIF()函数的使用
NULLIF(expr1,expr2)，如果expr1=expr2成立，那么返回值为null，否则返回值为expr1的值。

```
SELECT NULLIF('A','A');     -- 输出结果：null
SELECT NULLIF('A','B');     -- 输出结果：A
```

4、ISNULL()函数的使用
ISNULL(expr)，如果expr的值为null，则返回1，如果expr1的值不为null，则返回0。

```
SELECT ISNULL(NULL);        -- 输出结果：1
SELECT ISNULL('HELLO');     -- 输出结果：0
```

> [MySQL中IF()、IFNULL()、NULLIF()、ISNULL()函数的使用_ifnull mysql-CSDN博客](https://blog.csdn.net/pan_junbiao/article/details/85928004)

## DISTINCT 关键字

用于去重

[2356. 每位教师所教授的科目种类的数量 - 力扣（LeetCode）](https://leetcode.cn/problems/number-of-unique-subjects-taught-by-each-teacher/description/?envType=study-plan-v2&envId=sql-free-50)

## 开窗函数

之前还真不知道有这个东西。。

[【精选】三种开窗函数详细用法，图文详解-CSDN博客](https://blog.csdn.net/baomingshu/article/details/111945681)

其实就是我们有时候想用聚合函数，但是又想保留聚合的那些行。

- partition by:比如有A，B，C三个班级，我们想统计每个班级的平均值，但又想保留每个人的成绩。按照之前的做法肯定是先GROUP再JOIN，但是实际上可以使用开窗函数来完成这个操作。
- `OVER(order by [col])` 这个是按照列排序。

## CASE WHEN和IF子句

[Mysql：条件判断函数-CASE WHEN、IF、IFNULL详解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/292205172)

```SQL
CASE 字段 WHEN 预期值 THEN 结果1 ELSE 结果2 END
```

这不就是`IF(CONDITION, 结果1， 结果2)`吗

好像不同的点在于多条件时，可以做到很方便地判断

比如

```SQL
SELECT 
CASE 
        WHEN score<60 THEN "不及格"
        WHEN score>=60 and score<85 THEN "良"
        WHEN score>=85 THEN "优秀"
    ELSE "未知"
    END 
    AS "阶段"
,count(DISTINCT a.s_id) as "包含人数"
```



## 字符串函数

- SUBSTRING(column_name, start, length)：这将从列的值中提取一个子字符串，从指定的起始位置开始，直到指定的长度。

- UPPER(expression)：这会将字符串表达式转换为大写。

- LOWER(expression)：这会将字符串表达式转换为小写。

- CONCAT(string1, string2, ...)：这会将两个或多个字符串连接成一个字符串。

- ` GROUP_CONCAT`: 
  - [mysql中group_contact函数的使用_mysql group contact-CSDN博客](https://blog.csdn.net/xiao297328/article/details/120737556)

## 正则表达式

使用`REGEXP`查询

[正则表达式 – 教程 | 菜鸟教程 (runoob.com)](https://www.runoob.com/regexp/regexp-tutorial.html)