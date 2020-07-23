title: go-redis入门及踩坑日记
author: Tany
tags:
  - go
  - redis
categories:
  - redis
date: 2019-10-22 22:40:00
---
golang的非关系型数据库redis的踩坑实录

<!-- more -->

###### redis命令中的setex key value time,获取ttl
- 成功例：
``` 
totaltime := taskTimeOut + taskTimeOutErr  //预设时间+网络误差时间，初值10
    rds.SetNX("aatestNXTTL", "xxxxx", time.Duration(totaltime*1e9))
    ttl, _ := rds.TTL("aatestNXTTL").Result()
    for i := 0; i < 11; i++ {
        ttldecimal := ttl.Nanoseconds() / 1e9
        go func(ttldecimal int64) {
            fmt.Println(ttldecimal)
        }(ttldecimal)
        time.Sleep(time.Second)
        ttl, _ = rds.TTL("aatestNXTTL").Result()
    }
```
输出成功结果：
10 9 8 7 6 5 4 3 2 1 0
- 失败例:
```
rds.SetNX("aatestNXTTL", "x", time.Duration(totaltime)*time.Second)
```
输出失败结果：
 10 0 0 0 0 0 。。。。
 
****** 在 `time.Duration(totaltime)`中，totaltime默认的是 ns 纳秒。若输入`rds.SetNX("aatestNXTTL", "x", time.Duration(totaltime))` 如果反应快的话直接输出 `ttl`，不 `/1e9`转化成秒 ，可能会输出 10ns 。有时候反应慢了，就直接输出 0。

###### 创建redis-client 对象
```
client := redis.NewClient(&redis.Options{
        Addr:     "localhost:6379", //地址：端口
        Password: "", //密码没有可不填
        DB:       0, //库号
    })
```

###### 返回值
```
set 成功返回 "OK" 失败返回 "ERR syntax error"
hset 成功返回 "1" 失败返回 "err" 
```
###### redis简单操作
```
hmset 可以实现 n次 hset的操作
```

###### get存取值判断

-  Get() 取值 判断是否为空  val == "" 用空串比较
```
redis 存到期时间的值 id := "4001025061"
    rc.SetNX(id, "1", 10*time.Second) // key  value  timeout
    times2, err := rc.Get(id).Result()
    if err != nil {
        fmt.Println(err)
    }
    if times2 == "" {
        fmt.Println("值等于 nil")
    }
```