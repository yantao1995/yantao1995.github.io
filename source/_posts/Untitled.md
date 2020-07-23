title: redis底层实现之简单动态字符串 sds
author: Tany
tags:
  - redis
  - sds
categories:
  - redis
date: 2020-07-23 10:26:00
---
阅读redis书籍《redis设计与实现》笔记。
源码版本redis 3.0。

<!-- more -->

### SDS(简单动态字符串)

- 不直接使用char[]原因:
1. 杜绝缓冲区溢出。
2. 减少修改字符串时带来的内存重分配次数。
    - 字符串拼接append<防止缓冲区溢出>。
    - 字符串截断trim<防止内存泄露>。

#### 类似于golang切片，异同点：

sds 内部结构：
```
struct sdshdr {
    //记录buf数组中已使用的字节的数量
    //等于SDS所保存的字符串长度
    int len;
    //记录buf数组中未使用字节的数量
    int free;
    //字节数组，用于保存字符串
    char buf[];
}
```
golang 切片结构：
```
type slice struct {
    array unsafe.Pointer //引用切片底层的数组
    len int //用于限定可读写的元素数量
    cap int //表示切片所引用的数组的真实长度
}
```
 
- sds如果要计算容量cap： len+free 
- golang切片如果要计算未使用free： cap-len


#### 未使用空间的两种优化策略：空间预分配、惰性空间释放

##### 空间预分配

- 用于优化sds字符串增长操作，可以减少连续执行字符串增长操作所需的内存重分配次数。

策略：
 1. 修改后长度小于1MB时：分配的free和len同样大小。比如：修改后sds的len为13字节，那么free也会分配13。此时sds的buf数组长度为13+13+1=27byte。
 2. 修改后的长度大于1MB时：分配free为1MB。比如：修改后sds的len变成呃30MB，那么会分配1MB的未使用空间。此时sds的buf数组长度为 30mb+1mb+1byte 。

作用：sds将连续增长n次字符串所需的内存重分配次数从必定n次降低为最多n次。

##### 惰性空间释放

- 用于优化sds的字符串缩短操作。

策略：
 1. 当sds的api需要缩短sds保存的字符串时，并不立即内存重分配，而是使用free将这些字节数量记录起来，等待将来使用。

- sds也提供了真正释放sds的未使用空间的api，防止该策略造成内存浪费。


#### 二进制安全

与C字符串的比较：
- C: 字符必须符合某种编码，除末尾外不能包含空字符。所以无法保存图片，音频，视频...二进制数据。
- sds: 的api都会以处理二进制的方式来处理sds存放在buf数组的数据。
