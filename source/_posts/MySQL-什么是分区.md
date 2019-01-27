---
title: MySQL-什么是分区
date: 2019-2-26 17:42:41

tags:
- 性能优化
- MySQL

categories:
- 数据存储
- MySQL
---

# 概念
分区类似于水平分表。
水平分表需要用户预先手动显式创建出多张分表(如tbl_user0, tbl_user1, tbl_user2)，在物理上实实在在的创建多张表，通过客户端代理(Sharding-JDBC等)或者中间件代理(Mycat等)来实现分表逻辑。
分区是MySQL的一个插件Plugin功能，将一张大表的数据在数据库底层分成多个分区文件(如tbl_user#P#p0.ibd, tbl_user#P#p1.ibd, tbl_user#P#p2.ibd)，和水平分表不同的是分区不需要显式的创建“分表”,数据库会自动创建分区文件的，用户看到的只是一张普通的表，其实是对应的是多个分区，这个是对用户是屏蔽的、透明的，在使用上和使用一张表完全一样，不需要借助任何功能来实现。
分区是一种逻辑上的水平分表，在物理层面还是一张表。

# 分区类型
range分区：基于一个给定的连续区间范围(区间要求连续并且不能重叠)，把数据分配到不同的分区
- 注意最后一个区要设置在最大值以内，否则有部分数据没有专属分区，会导致bug

list分区：类似于range分区，区别在于list分区是居于枚举出的值列表分区，range是基于给定的连续区间范围分区
hash分区：基于给定的分区个数，把数据分配到不同的分区
key分区：类似于hash分区

# 弊端
分区表很帅气，为什么大部分互联网还是更多的选择自己分库分表来水平扩展咧？

回答：

1）分区表，分区键设计不太灵活，如果不走分区键，很容易出现全表锁

2）一旦数据量并发量上来，如果在分区表实施关联，就是一个灾难

3）自己分库分表，自己掌控业务场景与访问模式，可控。分区表，研发写了一个sql，都不确定mysql是怎么玩的，不太可控

4）运维的坑


**参考**
- https://blog.csdn.net/vbirdbest/article/details/82461109
- https://www.cnblogs.com/xiaoboluo768/p/5191964.html
- https://blog.csdn.net/laoyang360/article/details/52886987
- range分区的bug https://blog.csdn.net/cug_jiang126com/article/details/51984628
- MySQL 5.7.18 分区有bug https://www.cnblogs.com/liujiacai/p/7611071.html