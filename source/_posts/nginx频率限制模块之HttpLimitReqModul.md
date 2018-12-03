---
title: nginx频率限制模块之HttpLimitReqModul
date: 2018-11-30 12:26:41

tags:
- nginx
- 网络安全

categories:
- nginx

---

HttpLimitReqModul为nginx官方自带的**限制单位时间内请求次数**的功能模块，通过**limit_req_zone**和**limit_req**指令配合使用来达到限制。一旦对应的累计连接超过指定数量，就会返回503错误。
可以利用该规则对IP和URL进行访问频率的限制。

>- limit_conn_zone 定义IP或url访问的频率限制规则
只能配置在 http{} 
>- limit_conn 启用对应的规则
可以配置于http{}，server{}，location{} 

#### 限制规则
```
limit_req_zone $binary_remote_addr zone=ip_second:10m   rate=10r/s;
limit_req_zone $request_uri zone=url_minute:10m   rate=1r/m;

```
- limit_req_zone 可以理解为命令声明，类似于 `int x=0` 中的 int
##### 限制类型
- $binary_remote_addr 代表限制IP
- $request_uri 代表限制URL
##### 规则
- `zone=ip_second:10m   rate=10r/s`相当于`zone=限制规则变量名:内存占用上限 频率=次数r/秒`
##### 内存占用上限
- 1m大约可以存1.6W个ip回话
- 10m---16W
##### 频率限制
采用漏桶原理，简单理解就是，`rate=10r/s` 这十个请求平均分配在1秒内，也就是100ms最多能通过一次请求，而不是前面100ms接收10次，后面才不能接收。


#### 限制范围
当 limit_req 设置后，对应的 limit_req_zone 规则才会生效，而且只在 limit_req 声明的范围内生效
```
limit_req   zone=限制规则变量名 burst=5  nodelay;
```
##### 全局生效
将此命令放在 http{} 层，紧接着 limit_req_zone 之后
##### 局部生效
将此命令放在 server{} 或 location{} 层，则在此之外的层级不受影响，还可以在不同的层级设置不同的规则
```
server{
	limit_req   zone=限制规则变量名  burst=2  nodelay;
}
server{
	limit_req   zone=限制规则变量名  burst=5  nodelay;
}
```
##### burst
burst 代表令牌数量，至少要为1，即可以在limit_req_zone规则的基础上，额外可以请求的数量。
令牌消耗后，以 limit_req_zone 定义的时间单位恢复。
#### 频率单位
只能控制 秒（rate=100r/s） 和 分钟（rate=100r/m） 级别

#### 案例分析

多个限制时，只要有一个符合条件，就会触发限制

```
http {
        #限制ip
        limit_req_zone $binary_remote_addr zone=ip_second:10m   rate=10r/s;
        limit_req   zone=ip_second  burst=5  nodelay;
        limit_req_zone $binary_remote_addr zone=ip_minute:100m   rate=10r/m;
        limit_req   zone=ip_minute  burst=50  nodelay;

        #限制url
        limit_req_zone $request_uri zone=url_minute:10m   rate=1r/m;
        limit_req   zone=url_minute  burst=1  nodelay;

}
```
一秒内请求100次
>ab -n 100 -c 10 url

只会有51个成功，`rate=10r/m`代表一分钟10个请求，系统会默认把这十个请求平均分配在60秒内，即6秒一个请求，所以1秒内，只能用掉这里的1个名额，剩下50个名额用的是`burst=50`令牌的名额。



#### 自定义返回码
自定义限制规则生效后的返回码
```
http {
    limit_conn_status 503;

    limit_req_zone $binary_remote_addr zone=ip_second:10m   rate=100r/s;
    limit_req   zone=ip_second  burst=5  nodelay;
    limit_req_zone $binary_remote_addr zone=ip_minute:100m   rate=10r/m;
    limit_req   zone=ip_minute  burst=50  nodelay;
}
```



