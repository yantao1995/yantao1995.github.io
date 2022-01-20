title: redis底层实现之跳跃表
author: Tany
tags:
  - redis
  - zset
categories:
  - redis
date: 2020-09-20 17:17:00
---
阅读redis书籍《redis设计与实现》笔记。
源码版本redis 3.0。


<!-- more -->

### 跳跃表SkipList

- 源码3.0       redis.h

#### 2个地方使用

1. 有序集合的键
2. 集群节点中用作内部数据结构

#### 基本概念

- 有序
- 平均时间复杂度 O(logN),最坏O(N)

```
/*
 * 有序集合
 */
typedef struct zset {

    // 字典，键为成员，值为分值
    // 用于支持 O(1) 复杂度的按成员取分值操作
    dict *dict;

    // 跳跃表，按分值排序成员
    // 用于支持平均复杂度为 O(log N) 的按分值定位成员操作
    // 以及范围操作
    zskiplist *zsl;

} zset;
```

#### zskiplist

- 用于保存跳跃表节点相关信息

```
/*
 * 跳跃表
 */
typedef struct zskiplist {

    // 表头节点和表尾节点
    struct zskiplistNode *header, *tail;

    // 表中节点的数量
    unsigned long length;

    // 表中层数最大的节点的层数
    int level;

} zskiplist;
```

- header:指向表头节点。
- tail:指向表尾节点。
- level:记录目前跳跃表内，层数最大的节点的层数。 (不包含表头节点层数)
- length:记录跳跃表长度，即目前包含的节点数量。(不包含表头节点)

![zskiplist示例](http://redisbook.com/_images/graphviz-8fc5de396a5b52c3d0b1991a1e09558ad055dd86.png)

#### zskiplistNode

- 跳跃表节点

```
/* ZSETs use a specialized version of Skiplists */
/*
 * 跳跃表节点
 */
typedef struct zskiplistNode {

    // 成员对象
    robj *obj;

    // 分值
    double score;

    // 后退指针
    struct zskiplistNode *backward;

    // 层
    struct zskiplistLevel {

        // 前进指针
        struct zskiplistNode *forward;

        // 跨度
        unsigned int span;

    } level[];

} zskiplistNode;
```

- level（层）:

  1. 保存节点中的各层，有多少层数组就有多大。层内包含前进指针和跨度。(前进指针用于访问该层的下一个节点。跨度表示到下一个节点中间间隔了多少个节点)
  2. 层数根据幂次定律（越大的数出现的概率越小），随机生成1~32之间的值作为level数组大小。也就是高度。
- backward（后退指针）:指向当前节点的前一个节点。
- score（分值）:各个节点的排序依据，从小到大的分值排列。
- obj（成员对象）：

```
typedef struct redisObject {

    // 类型
    unsigned type:4;

    // 编码
    unsigned encoding:4;

    // 对象最后一次被访问的时间
    unsigned lru:REDIS_LRU_BITS; /* lru time (relative to server.lruclock) */

    // 引用计数
    int refcount;

    // 指向实际值的指针
    void *ptr;

} robj;
```

***表头节点和其他节点构造一样，但是主要保存层数，所以分值，成员对象，后退指针都不会用到***

- 迭代程序遍历时，从最高层，依次向下，选择到下一个节点跨度为1的层级，进入下一个节点直至null。