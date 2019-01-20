---
title: URL命名规范
date: 2018-12-25 22:02:41

tags:
- 代码规范

categories:
- 代码规范

---

## 请求方式
项目中统一只采用 get、post 两种方式
### get: 查询请求
参数不通过body，直接在放在URL上
### post: 涉及修改数据的请求
参数采用json格式，但URL也放上向get请求一样放上参数，用于请求日志查看、数据分析

## URL拼接方式
### 业务统一
第一个字符串代表具体的业务，如:
```buildoutcfg
/login/user/<int:user_id>
/login/user/<str:user_name>
/login/user/<int:phone>
```
login代表用户登陆的一系列请求。
这样做的好处一是统计的时候方便查看，二是万一要对某项业务做限制处理，只要用/login/去匹配就行。

### 内部统一

#### 根据内容查找弹幕
```buildoutcfg
`GET /barrage/lectures/get_barrages_by_content`
```
查找统一用search，需要查找的关键词用q作为参数传递
```buildoutcfg
`GET /barrage/lectures/search`
```
加上参数
```buildoutcfg
`/barrage/lectures/search?q=a`
```
#### 获取弹幕数量
```buildoutcfg
`GET /barrage/lectures/barrage_num`
```
数量统一用count表示
```buildoutcfg
`GET /barrage/lectures/barrage_count`
```
