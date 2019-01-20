---
title: 关于python的代码规范
date: 2018-12-25 22:02:41

tags:
- 代码规范

categories:
- 代码规范

---

为了代码的易读性，方便团队多人协作，请大家统一按此标准去规范自己的代码。

## PEP8
PEP8是 python 的代码规范标准，具体细节参考下面链接，在此不多叙述。
- 原文 https://www.python.org/dev/peps/pep-0008/
- 中文翻译 https://blog.csdn.net/ratsniper/article/details/78954852

可以用一些工具，如`atuopep8`去做辅助校验。

## 保持代码简洁
不用的代码，不管是废弃的功能，还是未来有用，但目前用不上的，都删除。
如果需要，再从git版本记录中找回。
lib中的统一工具除外。

## 内部命名规范
- 数量 count
- 开始时间 begin_time
- 结束时间 end_time
- 分页，起始值 offset，数量 limit
- 关键字查找，参数为q，代码中的变量为key_word



## sql语句
sql语句被包装成字符串，一般工具无法对此进行格式化校验，但仍需遵循以下规则：

### 关键字大写
```buildoutcfg
select id,name from test_barrage where id = 3;
```
修正为：

```buildoutcfg
SELECT id,name FROM test_barrage WHERE id = 3;
```

### 缩进
```buildoutcfg
SELECT id,name FROM test_barrage WHERE id = 3 and type=2 ORDER BY id;
```
错误点：
1. `type=2`空格前后应有空格`type = 2`
1. sql语句过长，`SELECT`、`WHERE`、`ORDER BY` 需要一行一个
1. `WHERE`条件一行一个

修正为：

```buildoutcfg
SELECT id,name FROM test_barrage 
WHERE id = 3 
    and type=2 
ORDER BY id;
```

### 慎用*
```buildoutcfg
select * from test_barrage where id = 3;
```
如果不是必要，修正为：

```buildoutcfg
SELECT id,name FROM test_barrage WHERE id = 3;
```

### 参数化
为了防止sql注入，pymysql提供了两种参数校验方式
1. 列表传参
```buildoutcfg
sql = "SELECT `id`, `password` 
        FROM `users` 
        WHERE `email`=%s"
cursor.execute(sql, ['webmaster@python.org'])
```
2. 字典传参
```buildoutcfg
sql = "SELECT `id`, `password` 
        FROM `users` 
        WHERE `email`=%(email)s"
cursor.execute(sql, {"email":'webmaster@python.org'})
```
第一种比较容易出错，统一采用第二种

## pylint
代码提交前用 pylint 进行对代码规范进行打分，9.5以上方为合格。具体使用方法请参考：
- https://pypi.org/project/pylint/
- https://www.cnblogs.com/zhangningyang/p/8652941.html
- https://www.ibm.com/developerworks/cn/linux/l-cn-pylint/index.html


