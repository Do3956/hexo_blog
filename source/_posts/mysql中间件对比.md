---
title: MySQL-分区及中间件简介
date: 2019-2-26 17:42:41

tags:
- 大数据
- MySQL

categories:
- 数据存储
- MySQL
---

## 中间件

分类
- lib库
    - 业务直接到数据库，少一层proxy效率更高
    - 没有proxy的lvs的单点问题

- proxy
    - 统一管理所有到数据库的连接，连接复用
    - 基础查询功能抽象，减少代码耦合
    - 易于实现监控、数据迁移、连接管理等功能


### sharding-jdbc（开源，lib）
> 当当应用框架ddframe中，从关系型数据库模块dd-rdb中分离出来的数据库水平分片框架
可惜是Java写的
#### 功能
1）以jar包形式提供服务

2）分片灵活，支持等号、between、in等多维度分片

3）sql解析，支持聚合、分组、排序、limit、or等

#### 特点

1）分片规则：策略自定义、复数分片数

2）JDBC规范重写：针对DataSource、Connection、Statement、PreparedStatement和ResultSet接口封装

3）sql解析：druid

4）sql改写：修改逻辑表名->真实表名；替换不支持的功能，如：avg->sum和count

5）sql路由：单表路由、binding表路由、笛卡尔积路由

6）sql执行：多线程并发执行sql

7）结果归并：遍历类、排序类（归并排序）、聚合类（比较型、累加型、平均型）、分组类

### NewSQL
https://baijiahao.baidu.com/s?id=1592030931074047993&wfr=spider&for=pc

### mycat（开源，proxy）
> 社区爱好者在阿里cobar基础上进行二次开发，解决了cobar的一些问题，且加入了新的功能。
不适用场景
https://www.sohu.com/a/211953442_470018

### MaxScale
> mariadb (MySQL原作者维护的一个版本)研发，目前版本不支持分库分表。

### MySQL Route
> MySQL 官方Oracle公司发布出来的一个中间件

### Atlas
> 360团队基于 mysql proxy 把lua用c改写，在高并发下经常会挂掉。

### DBproxy（开源，proxy）
> 针对atlas进行改进，形成了新的高可靠、高可用企业级数据库中间件DBProxy

### oneproxy（不开源，proxy）
> 基于mysql协议的数据库中间件。利用c进行开发的，专注于性能和稳定性。

### vitess
> Youtube上产使用的，架构复杂，不支持MySQL原生协议

### TDDL
> Tabao开发，主要解决了分库分表对应用的透明化以及异构数据库之间的数据复制。
复杂度相对较高。当前公布的文档较少，只开源动态数据源，分表分库部分还未开源，还需要依赖diamond，不推荐使用。

### DBProxy
> 美团点评基于 Atlas 的改进。
其特性主要有：读写分离、负载均衡、支持分表、IP过滤、sql语句黑名单、DBA平滑下线DB、从库流量配置、动态加载配置项







**参考**
- https://www.cnblogs.com/mliudong/p/3625522.html
- https://www.cnblogs.com/taoxinrui/p/6782791.html

