---
title: 从SQL的执行原理优化多表查询
date: 2019-2-26 17:42:41

tags:
- MySQL
- 性能优化

categories:
- 数据存储
- MySQL
---


### 查询sql语句的执行流程
根据这个顺序，将可优化的点往前放，尽早降低后面的处理次数

```
From - 将涉及的表加载到内存

ON - 作为产生笛卡尔积表的挑选条件

JOIN - 此处产生笛卡尔积表（A表*B表），需要大量内存

WHERE - 数据进行过滤，取出符合条件的记录行，生成一张临时表

GROUP BY - 将上一步的临时表切分成若干临时表

SELECT - 如果有GROUP BY，会对N个临时表进行N次查询，最终形成一个新的临时表

HAVING - 对SELECT 语句执行之后的临时表中的数据过滤

ORDER BY

LIMIT
```
> 当两张表的数据量比较大，又需要连接查询时，应该使用 `FROM table1 JOIN table2 ON xxx` 的语法，避免使用 `FROM table1,table2 WHERE xxx` 的语法，因为后者会在内存中先生成一张数据量比较大的笛卡尔积表，增加了内存的开销。


### 计算SQL平均耗时
计算 sql查询所耗费的平均时间时，要关掉mysql的查询缓存，防止多次执行从缓存中读取数据
可以用 `SQL_NO_CACHE`
```
SELECT
    SQL_NO_CACHE *
FROM
    mm_member
```

### 慎用子查询
mm_member 为用户表（370 行），mm_log 为登陆记录（15W 行）
现在要求我们根据这两张表查出2017-02-06那天有那些用户登录过系统。

#### join
```
SELECT
    SQL_NO_CACHE mm.*
FROM
    mm_member mm
JOIN
    mm_log ml
ON
    mm.id = ml.member_id
WHERE
    ml.access_time LIKE '%2017-02-06%'
GROUP BY
    ml.member_id;
```
运行次数 mm_member * mm_log， 但是在on这一步就把后面的运行次数降到了 mm_member 的级别。

耗时：0.120s

#### 子查询 in

```
SELECT
    SQL_NO_CACHE mm.username
FROM
    mm_member mm
WHERE
    mm.id IN (SELECT ml.member_id FROM mm_log ml WHERE ml.access_time LIKE '%2017-02-06%' GROUP BY ml.member_id);
```
运行次数 mm_member * mm_log， 和我们以为的不同，这里 IN 后面的子查询，是每一行mm_member都会去重新执行一遍，并且没有用到缓存。

耗时：40.834s > 0.120s*370=44.4

其中子查询语句：
```
SELECT SQL_NO_CACHE member_id FROM mm_log ml WHERE ml.access_time LIKE '%2017-02-06%' GROUP BY ml.member_id;
```
仅耗时 0.11s


#### 子查询 exist
```
SELECT
    SQL_NO_CACHE mm.*
FROM
    mm_member mm
WHERE
    EXISTS (SELECT * FROM mm_log ml WHERE mm.id = ml.member_id AND ml.access_time LIKE '%2017-02-06%');
```
相比于IN，这里的优化在于子查询，多了`mm.id = ml.member_id`条件，在where处就把不符合条件的记录过滤掉。

耗时：34s

对于在子查询中使用IN的性能高还是是用EXITS的性能高，有一种普遍的说法是：

1，在外表大，内表小，外表中有索引的情况下，使用IN。

2，在外表小，内表大，内表中有索引的情况下，使用EXITS。


**参考**

感谢大神分享，本文更多是总结和自己的理解补充，内容和例子都来自于
- sql语句的执行流程 https://blog.csdn.net/csdn_hklm/article/details/78394412
- 笛卡尔积表 http://www.cnblogs.com/cdf-opensource-007/p/6502556.html
- 慎用子查询 https://www.cnblogs.com/cdf-opensource-007/p/6540521.html
