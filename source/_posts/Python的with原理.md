---
title: Python的with原理
date: 2018-12-3 17:02:41

tags:
- Python

categories:
- Python

---
### 引言

对一个文件进行操作，我们可以这样写：
```
f = open(file, mode)
...
f.close() #关闭文件
```
但是可能会在文件操作的过程中触发异常，导致`f.close()`没有被执行，于是我们对其做出以下改良：
```
f = open(file, mode)
try:
    ...
finally:
    f.close()
```

嗯...貌似有点繁琐，能不能用一个简单点的方法呢？这时`with`就出现了：
```
with open(file, mode) as f:
    ...
```
相当于用with包裹后，自动处理了文件的关闭。

### 概念

那具体是怎么实现的呢？
我们先来看看有关上下文管理器的概念：

- 上下文管理协议（Context Management Protocol）
> 包含方法 __enter__() 和 __exit__()，支持
该协议的对象要实现这两个方法。

- 上下文管理器（Context Manager）
> 支持上下文管理协议的对象，这种对象实现了
__enter__() 和 __exit__() 方法。上下文管理器定义执行 with 语句时要建立的运行时上下文，
负责执行 with 语句块上下文中的进入与退出操作。通常使用 with 语句调用上下文管理器，
也可以通过直接调用其方法来使用。

- 运行时上下文（runtime context）
> 由上下文管理器创建，通过上下文管理器的 __enter__() 和
__exit__() 方法实现，__enter__() 方法在语句体执行之前进入运行时上下文，__exit__() 在
语句体执行完后从运行时上下文退出。with 语句支持运行时上下文这一概念。

### with 语句执行过程
```
context_manager = context_expression
exit = type(context_manager).__exit__ 
value = type(context_manager).__enter__(context_manager)
exc = True   # True 表示正常执行，即便有异常也忽略；False 表示重新抛出异常，需要对异常进行处理
try:
    try:
        target = value  # 如果使用了 as 子句
        with-body     # 执行 with-body
    except:
        # 执行过程中有异常发生
        exc = False
        # 如果 __exit__ 返回 True，则异常被忽略；如果返回 False，则重新抛出异常
        # 由外层代码对异常进行处理
        if not exit(context_manager, *sys.exc_info()):
            raise
finally:
    # 正常退出，或者通过 statement-body 中的 break/continue/return 语句退出
    # 或者忽略异常退出
    if exc:
        exit(context_manager, None, None, None)
    # 缺省返回 None，None 在布尔上下文中看做是 False

```
### 自定义上下文管理器
现在试着自己实现一个上下文管理器，实现对文件资源的释放
```
#/usr/bin/python
# -*- coding: utf-8 -*-
# 自行定義 Context Manager
class File(object):
  def __init__(self, filename, mode):
    # 設定檔名與開檔模式
    self.filename = filename
    self.mode = mode

  # 配給資源（開啟檔案）
  def __enter__(self):
    print("開啟檔案：" + self.filename)
    self.open_file = open(self.filename, self.mode)
    return self.open_file

  # 回收資源（關閉檔案）
  def __exit__(self, type, value, traceback):
    print("關閉檔案：" + self.filename)
    self.open_file.close()


with File("file.txt", "w") as f:
  print("寫入檔案...")
  f.write("Hello, world.")
```

另一种实现方式
```
#/usr/bin/python
# -*- coding: utf-8 -*-
from contextlib import contextmanager

# 自行定義 Context Manager
@contextmanager
def open_file(name, mode):
  # 配給資源（開啟檔案）
  f = open(name, mode)
  yield f
  # 回收資源（關閉檔案）
  f.close()

with open_file('file.txt', 'w') as f:
  f.write("Hello, world.")
```

### 对with的总结
> with适用于对资源进行访问的场合，确保不管使用过程中是否发生异常都会执行必要的“清理”操作，释放资源，比如文件使用后自动关闭、线程中锁的自动获取和释放等。

### nested 同时使用多个with嵌套
```
with nested(A(), B(), C()) as (X, Y, Z):
     # with-body code here
```
```
with A() as X:
    with B() as Y:
        with C() as Z:
             # with-body code here
```
需要注意的是，发生异常后，如果某个上下文管理器的 __exit__() 方法对异常处理返回 False，则更外层的上下文管理器不会监测到异常。


参考
- https://www.ibm.com/developerworks/cn/opensource/os-cn-pythonwith/index.html
- https://blog.gtwang.org/programming/python-with-context-manager-tutorial/