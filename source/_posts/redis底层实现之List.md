title: redis底层实现之List
author: Tany
tags:
  - redis
  - list
categories:
  - redis
date: 2020-08-24 15:51:00
---
阅读redis书籍《redis设计与实现》笔记。
源码版本redis 3.0。
对比quickList的源码版本 redis 6.0.5。

<!-- more -->

### List列表

#### 版本3.2之前，list使用两种数据结构作为底层实现：

1. 压缩列表ziplist （特殊的双向链表，没有指针，靠entry推算位置）
2. 双向链表adlist (list)

- 因为双向链表占用的内存比压缩列表要多， 所以当创建新的列表键时， 列表会优先考虑使用压缩列表， 并且在有需要的时候， 才从压缩列表实现转换到双向链表实现。

##### ziplist 

- 为了节约内存,没有维护双向指针:prev next,而是存储上一个 entry的长度和 当前entry的长度，通过长度推算下一个元素在什么地方。（牺牲读取的性能，获得高效的存储空间，指针比entry费空间<时间换空间>）
- 字段、值比较小时使用。
- entry 采用变长编码。

- zlentry 源码结构：
```
//zlentry(entry)结构:
typedef struct zlentry {

    // prevrawlen ：前置节点的长度
    // prevrawlensize ：编码 prevrawlen 所需的字节大小
    unsigned int prevrawlensize, prevrawlen;

    // len ：当前节点值的长度
    // lensize ：编码 len 所需的字节大小
    unsigned int lensize, len;

    // 当前节点 header 的大小
    // 等于 prevrawlensize + lensize
    unsigned int headersize;

    // 当前节点值所使用的编码类型
    unsigned char encoding;

    // 指向当前节点的指针
    unsigned char *p;

} zlentry;
```
- 《redis设计与实现》书中配套源码注释对zip结构解析示例:

```
/* 
空白 ziplist 示例图

area        |<---- ziplist header ---->|<-- end -->|

size          4 bytes   4 bytes 2 bytes  1 byte
            +---------+--------+-------+-----------+
component   | zlbytes | zltail | zllen | zlend     |
            |         |        |       |           |
value       |  1011   |  1010  |   0   | 1111 1111 |
            +---------+--------+-------+-----------+
                                       ^
                                       |
                               ZIPLIST_ENTRY_HEAD
                                       &
address                        ZIPLIST_ENTRY_TAIL
                                       &
                               ZIPLIST_ENTRY_END

非空 ziplist 示例图

area        |<---- ziplist header ---->|<----------- entries ------------->|<-end->|

size          4 bytes  4 bytes  2 bytes    ?        ?        ?        ?     1 byte
            +---------+--------+-------+--------+--------+--------+--------+-------+
component   | zlbytes | zltail | zllen | entry1 | entry2 |  ...   | entryN | zlend |
            +---------+--------+-------+--------+--------+--------+--------+-------+
                                       ^                          ^        ^
address                                |                          |        |
                                ZIPLIST_ENTRY_HEAD                |   ZIPLIST_ENTRY_END
                                                                  |
                                                        ZIPLIST_ENTRY_TAIL
*/
```

- 连锁更新问题：

##### adlist 

- adlist是标准的双向链表，Node节点包含prev和next指针，可以双向遍历；保存了 head 和 tail 指针，表头和表尾进行插入的复杂度都为 (1) 

- 源码

```
/*
 * 双端链表节点
 */
typedef struct listNode {

    // 前置节点
    struct listNode *prev;

    // 后置节点
    struct listNode *next;

    // 节点的值
    void *value;

} listNode;

/*
 * 双端链表结构
 */
typedef struct list {

    // 表头节点
    listNode *head;

    // 表尾节点
    listNode *tail;

    // 节点值复制函数
    void *(*dup)(void *ptr);

    // 节点值释放函数
    void (*free)(void *ptr);

    // 节点值对比函数
    int (*match)(void *ptr, void *key);

    // 链表所包含的节点数量
    unsigned long len;

} list;
```

- 无环： 表头结点的prve和表尾结点的next指向 null。
- 带链表长度计数器：len 记录了结点数量。
- 多态：链表使用void*指针来保存值，并且包含 dup,free,match属性来实现特定功能函数。

##### ziplist -> adlist 转化条件

1. 试图往列表新添加一个字符串值，且这个字符串的长度超过 server.list_max_ziplist_value （默认值为 64 ）
2. ziplist 包含的节点超过 server.list_max_ziplist_entries （默认值为 512 ）

- 在redis.conf中可以修改
```
list-max-ziplist-value 64 
list-max-ziplist-entries 512 
```

#### 3.2+ 之后，list使用 quickList 作为底层实现:

- 结合了 adlist和ziplist的优点。

- redis6.0.5 源码

```

/* Node, quicklist, and Iterator are the only data structures used currently. */

/* quicklistNode is a 32 byte struct describing a ziplist for a quicklist.
 * We use bit fields keep the quicklistNode at 32 bytes.
 * count: 16 bits, max 65536 (max zl bytes is 65k, so max count actually < 32k).
 * encoding: 2 bits, RAW=1, LZF=2.
 * container: 2 bits, NONE=1, ZIPLIST=2.
 * recompress: 1 bit, bool, true if node is temporarry decompressed for usage.
 * attempted_compress: 1 bit, boolean, used for verifying during testing.
 * extra: 10 bits, free for future use; pads out the remainder of 32 bits */
typedef struct quicklistNode {
    struct quicklistNode *prev;
    struct quicklistNode *next;
    unsigned char *zl;          //保存的数据 压缩前ziplist 压缩后压缩的数据
    unsigned int sz;             /* ziplist size in bytes */
    unsigned int count : 16;     /* count of items in ziplist */
    unsigned int encoding : 2;   /* RAW==1 or LZF==2 */
    unsigned int container : 2;  /* NONE==1 or ZIPLIST==2 */
    unsigned int recompress : 1; /* was this node previous compressed? */
    unsigned int attempted_compress : 1; /* node can't compress; too small */
    unsigned int extra : 10; /* more bits to steal for future usage */
} quicklistNode;


/* quicklist is a 40 byte struct (on 64-bit systems) describing a quicklist.
 * 'count' is the number of total entries.
 * 'len' is the number of quicklist nodes.
 * 'compress' is: -1 if compression disabled, otherwise it's the number
 *                of quicklistNodes to leave uncompressed at ends of quicklist.
 * 'fill' is the user-requested (or default) fill factor.
 * 'bookmakrs are an optional feature that is used by realloc this struct,
 *      so that they don't consume memory when not used. */

typedef struct quicklist {
    quicklistNode *head;
    quicklistNode *tail;
    unsigned long count;        /* total count of all entries in all ziplists */
    unsigned long len;          /* number of quicklistNodes */
    int fill : QL_FILL_BITS;      //#   define QL_FILL_BITS 14        /* fill factor for individual nodes */
    unsigned int compress : QL_COMP_BITS; //#   define QL_COMP_BITS 14 /* depth of end nodes not to compress;0=off */
    unsigned int bookmark_count: QL_BM_BITS;//#   define QL_BM_BITS 4
    quicklistBookmark bookmarks[];
} quicklist;


/* Bookmarks are padded with realloc at the end of of the quicklist struct.
 * They should only be used for very big lists if thousands of nodes were the
 * excess memory usage is negligible, and there's a real need to iterate on them
 * in portions.
 * When not used, they don't add any memory overhead, but when used and then
 * deleted, some overhead remains (to avoid resonance).
 * The number of bookmarks used should be kept to minimum since it also adds
 * overhead on node deletion (searching for a bookmark to update). */
typedef struct quicklistBookmark {
    quicklistNode *node;
    char *name;
} quicklistBookmark;

```

- quickList也是标准双向链表，有head，tail
-  每一个quicklistNode 包含 一个ziplist，*zl 压缩链表里存储键值。所以quicklist是对ziplist进行一次封装，使用小块的ziplist来既保证了少使用内存，也保证了性能。