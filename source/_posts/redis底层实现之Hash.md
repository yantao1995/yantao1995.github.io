title: redis底层实现之Hash
author: Tany
tags:
  - redis
  - hash
categories:
  - redis
date: 2020-09-02 12:42:00
---
阅读redis书籍《redis设计与实现》笔记。
源码版本redis 3.0。


<!-- more -->

### Hash

- 源码3.0       dict.h

#### 哈希表与哈希表节点

- 定义：

```
/*
 * 哈希表
 *
 * 每个字典都使用两个哈希表，从而实现渐进式 rehash 。
 */
typedef struct dictht {
    
    // 哈希表数组
    dictEntry **table;

    // 哈希表大小
    unsigned long size;
    
    // 哈希表大小掩码，用于计算索引值
    // 总是等于 size - 1
    unsigned long sizemask;

    // 该哈希表已有节点的数量
    unsigned long used;

} dictht;

/*
 * 哈希表节点
 */
typedef struct dictEntry {
    
    // 键
    void *key;

    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;

    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;

} dictEntry;
```
- union简单理解:共用体,共用一个内存首地址，并且各种变量名都可以同时使用，操作也是共同生效。如此多的access内存手段，确实好用，不过这些“手段”之间却没法互相屏蔽——就好像数组+下标和指针+偏移一样。 （极致的内存压榨）

- table:数组中每个元素都是指向dictEntry节点的指针。

- dictEntry 保存键值对，v使用union，可以是指针，也可以是uint64_t或者int64_t。（类型_t 是因为是 typedef 定义的，而不是新类型，原因是因为跨平台）

#### 字典

- dict定义：

```
/*
 * 字典
 */
typedef struct dict {

    // 类型特定函数
    dictType *type;

    // 私有数据
    void *privdata;

    // 哈希表
    dictht ht[2];

    // rehash 索引
    // 当 rehash 不在进行时，值为 -1
    int rehashidx; /* rehashing not in progress if rehashidx == -1 */

    // 目前正在运行的安全迭代器的数量
    int iterators; /* number of iterators currently running */

} dict;

```

- type 和 privdata 指针类型，为了创建多态字典。
- type 保存用于操作特定类型键值对的函数。
- privdata 保存了需要穿给那些类型特定函数的可选参数。

------

- dictType定义:

```
/*
 * 字典类型特定函数
 */
typedef struct dictType {

    // 计算哈希值的函数
    unsigned int (*hashFunction)(const void *key);

    // 复制键的函数
    void *(*keyDup)(void *privdata, const void *key);

    // 复制值的函数
    void *(*valDup)(void *privdata, const void *obj);

    // 对比键的函数
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);

    // 销毁键的函数
    void (*keyDestructor)(void *privdata, void *key);
    
    // 销毁值的函数
    void (*valDestructor)(void *privdata, void *obj);

} dictType;
```

- 一般状态下的结构：

1. dict 的 ht 指向 2个 dictht，
2. dictht的table指向 dictEntry*[]
3. dictEntry*[] 存储 dictEndty的键值对key,value节点。

```
dict.ht  -> dictht[]
dictht.table -> dictEntry*[]
dictEntry{key,value}
```

![字典结构图](http://redisbook.com/_images/graphviz-93608325578e8e45848938ef420115bf2227639e.png)

#### 解决冲突

- 链地址法：
    1. 每个哈希表节点都有一个next指针，多个哈希表节点可以用next指针构成一个单向链表，被分配到同一个索引上的多个节点可以用这个单向链表连接起来，解决了键冲突的问题。
    2. 新节点添加在链表表头位置。（为了速度）

#### rehash条件：扩容和收缩(依赖负载因子)

- 哈希表负载因子计算公式：

`load_factory = ht[0].used / ht[0].size (负载因子 = 哈希表已保存节点数量 / 哈希表大小)`

- 扩容

    1. 程序没有执行BGSAVE命令或者BGREWRITEAOF(AOF重写)命令，并且哈希表的负载因子大于等于1

    2. 如果程序正在执行BGSAVE或者BGREWRITEAOF(AOF重写)命令并且哈希表的负载因子大于等于5。在执行RDB或者AOF重写操作时，redis会创建当前服务器的子进程执行相应操作，为了避免在子进程存在期间对哈希表进行扩展操作，将扩展因子提高。可以避RDB或者AOF重写时不必要的内存写入操作，最大限度的节约内存。

- 收缩：负载因子小于0.1进行收缩

#### 渐进式rehash

- 概念：扩展或收缩哈希表时 ht[0] 的键值对 rehash到  ht[1] 不是一次完成，而是分多次，渐进式的。

- 原因：键值对数据量庞大时，一次性rehash的计算量可能会导致服务器在一段时间内停止服务。

- 步骤：
    1. 为 ht[1] 分配空间， 让字典同时持有 ht[0] 和 ht[1] 两个哈希表。
    2. 在字典中维持一个索引计数器变量 rehashidx ， 并将它的值设置为 0 ， 表示 rehash 工作正式开始。
    3. 在 rehash 进行期间， 每次对字典执行添加、删除、查找或者更新操作时， 程序除了执行指定的操作以外， 还会顺带将 ht[0] 哈希表在 rehashidx 索引上的所有键值对 rehash 到 ht[1] ， 当 rehash 工作完成之后， 程序将 rehashidx 属性的值增一。
    4. 随着字典操作的不断执行， 最终在某个时间点上， ht[0] 的所有键值对都会被 rehash 至 ht[1] ， 这时程序将 rehashidx 属性的值设为 -1 ， 表示 rehash 操作已完成。

- 过程中：
    1. 删除，查找，更新会同时在两个哈希表上进行。查找顺序：ht[0] -> ht[1] 。
    2. 添加会在ht[1]上进行。保证了ht[0]只减不增，随着rehash操作最终变成空表。