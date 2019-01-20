---
title: 聊一聊flask的get
date: 2018-12-20 22:02:41

tags:
- flask

categories:
- Python
- flask

---

## 获取参数的方式

#### 1. 通过的URL获取，如：zoo_id，cat_id
#### 2. 通过 request.args.get 获取，如：name
```buildoutcfg

@BP.route('/zoo/<int:zoo_id>/cat/<int:cat_id>', methods=['GET'])
def get_cat_info(zoo_id, cat_id):
    """  """
    name = request.args.get('name', type=str, default=None)
```


## get 获取json格式
#### 1. 通过URL传递
这种方法虽然比较麻烦，但是接口较为通用
```buildoutcfg
@BP.route('/zoo/<int:zoo_id>/cats', methods=['GET'])
def get_cat_list(zoo_id):
    """  """
ids = request.args.get('ids', type=str, default=0)
ids = json.loads(ids)
```
测试方法
```buildoutcfg
url = '/zoo/<int:zoo_id>/cats?ids=%s' % json.dumps([1,2,3])
request.get(url)
```

#### 2. 通过body传递
get_json比较简单，获取到的已经是json格式了，可以直接使用。
但get一般都不会通过body来传参，非一般规范，可以和项目协商后，统一采用一种方式。
```buildoutcfg
@BP.route('/zoo/<int:zoo_id>/cats', methods=['GET'])
def get_cat_list(zoo_id):
params = request.get_json()
id = params.get("id")
```
测试方法
```buildoutcfg
url = '/zoo/<int:zoo_id>/cats'
request.get(url, json={"ids":[20]})
```