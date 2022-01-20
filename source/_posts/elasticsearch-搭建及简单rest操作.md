title: elasticsearch 搭建及简单rest操作
author: Tany
tags:
  - es
  - 笔记
categories:
  - es
date: 2022-01-19 18:54:00
---
学习 elasticsearch 的简单笔记，记不住了就来看看。

<!-- more -->

## ES部署 

- host文件ip地址映射

```
vim /etc/hosts

192.168.25.129 node1
192.168.25.130 node2
192.168.25.131 node3
```


### 单机部署
- 不能使用root账户，新建账户，赋予权限运行

```
adduser elasticsearch  //添加用户
passwd elasticsearch    //创建密码
chown -R elasticsearch elasticsearch_path //赋予文件夹权限
su elasticsearch //切换用户
./elasticsearch_path/bin/./elasticsearch -d  //后台启动

```

- 启动后只能 localhost:9200 访问，需要ip访问则要修改 elasticsearch.yml 文件. 才可以 ip:9200访问

```
network.host: 本机ip
cluster.initial_master_nodes: ["本机ip"]
```

- 启动后若提示虚拟内存过小 

```
// 永久生效

vim /etc/security/limits.conf

* soft nofile 65535
* hard nofile 65535
* soft nproc 4096
* hard nproc 4096

其中nofile : 一个进程最多能打开的的文件数
其中nproc : 一个用户最多能创建的进程数
其中  * 代表所有用户，也可以直接使用指定用户名


//或者临时生效
ulimit -u 4096

```

- 启动后若提示当前用户最大内存过小

```

//打开文件
vim /etc/sysctl.conf
//添加
vm.max_map_count=262144

//执行命令，立即生效
sysctl -p

```


### 集群配置

- elasticsearch.yml

```

#集群名称
cluster.name: cluster-es  #当前集群内所有主机的名称应该一致
#节点名称 不能重复  ----此处需要修改
node.name: node1   
#ip地址   ---- 此处需要修改
network.host: node1
#是不是有资格主节点
node.master: true
node.data: true
http.port: 9200
#head 插件需要的配置
http.cors.allow-origin: "*"
http.cors.enabled: true
http.max_content_length: 200mb
#es7 之后新增的，用于初始化新集群时来选举leader
cluster.initial_master_nodes: ["node1","node2","node3"]
#es7 之后新增的，节点发现  ----- 此处需要修改
discovery.seed_hosts: ["node1:9300","node2:9300","node3:9300"]
network.tcp.keep_alive: true
network.tcp.no_delay: true
transport.tcp.compress: true
#集群内同时启动的数据任务个数，默认2
cluster.routing.allocation.cluster_concurrent_rebalance: 16
#添加或删除节点及负载均衡时并发恢复的线程个数,默认4
cluster.routing.allocation.node_concurrent_recoveries: 16
#初始化数据恢复时，并发恢复线程的个数，默认4
cluster.routing.allocation.node_initial_primaries_recoveries: 16

```



## 索引  

- 相当于 mysql 中的一个库

```
### 创建索引
PUT  http://192.168.25.128:9200/shopping

#### 获取单个索引
GET  http://192.168.25.128:9200/shopping

#### 获取全部索引
GET  http://192.168.25.128:9200/_cat/indices?v

#### 删除索引
DELETE   http://192.168.25.128:9200/shopping
```

## 文档 

- 相当于 mysql 中的行数据

```

### 创建文档
POST   http://192.168.25.128:9200/shopping/_doc
Content-Type: application/json

{
    "title": "小米手机",
    "category": "小米",
    "images": "asdasd/xm.jpg",
    "price": 399.00
}

### 创建文档 指定id
# 指定id http://192.168.25.128:9200/shopping/_doc/id1002
# 指定创建 http://192.168.25.128:9200/shopping/_create/id1002
PUT   http://192.168.25.128:9200/shopping/_create/id1004
Content-Type: application/json

{
    "title": "小米手机",
    "category": "小米",
    "images": "asdasd/xm.jpg",
    "price": 399.00
}


#### 主键查询
#全部查询 
GET  http://192.168.25.128:9200/shopping/_search
###单个查询
GET  http://192.168.25.128:9200/shopping/_doc/id1002

### 全量更新
PUT   http://192.168.25.128:9200/shopping/_doc/id1002
Content-Type: application/json

{
    "title": "小米手机1",
    "category": "小米1",
    "images": "asdasd/xm.jpg",
    "price": 10.00
}

### 局部更新
POST    http://192.168.25.128:9200/shopping/_update/id1003
Content-Type: application/json

{
    "doc":{
        "title":"华为手机",
        "price": 20
    }
}

###删除
DELETE   http://192.168.25.128:9200/shopping/_doc/id1002


###### 高级查询

###条件查询
GET  http://192.168.25.128:9200/shopping/_search
Content-Type:  application/json

{
    "query":{
        "match":{
            "category":"小米"
        }
    }
}

###条件查询 全量
GET  http://192.168.25.128:9200/shopping/_search
Content-Type:  application/json

{
    "query":{
        "match_all":{
        }
    }
}

###对字符串字段排序,聚合 需要修改
### url说明: ip:port/索引/_mapping?pretty 
PUT   http://192.168.25.128:9200/shopping/_mapping?pretty 
Content-Type: application/json

{
  "properties": {
    "price": { //需要操作的字段，
      "type":     "text",
      "fielddata": true
    }
  }
}

###条件查询 分页排序筛选
GET  http://192.168.25.128:9200/shopping/_search
Content-Type:  application/json

{
    "query":{
        "match_all":{
        }
    },
    "from":1, //offset
    "size":2, //limit
    "_source":["title"], //只需要的数据，不填就查全部 
    "sort":{  //排序
        "price":{
            "order":"desc"
        }
    } 
}

###条件查询 条件匹配
GET  http://192.168.25.128:9200/shopping/_search
Content-Type:  application/json

{
    "query":{
        "bool":{  //条件
            "must":[  // must 多个条件同时成立  ， should 多个条件任意成立
                {
                    "match":{ //只要包含有就匹配
                        "category":"小米"
                    }
                },
                {
                    "match_phrase":{ //全词匹配
                        "price":200
                    }
                }
            ],
            "filter":{ //过滤
                "range" :{ //范围
                    "price":{  
                        "gt":20  //大于20
                    }
                }
            }
        }
    },
    "highlight":{ //高亮显示，该字段增加html样式
        "fields":{
            "category":{}
        }
    }
}



###条件查询 聚合操作
GET  http://192.168.25.128:9200/shopping/_search
Content-Type:  application/json

{
    "aggs":{ //聚合操作
        "price_group":{ //名称，随意起名
            "terms":{ // 具体操作  term分组  avg平均值
                "field":"price" //分组字段
            }
        }
    },
    "size":1
}
```

## 映射

- 索引的映射相当于数据库中的表结构，对数据结构进行定义 

```

###创建user index
PUT  http://192.168.25.128:9200/user
###添加
PUT  http://192.168.25.128:9200/user/_mapping
Content-Type:  application/json

{
    "properties":{
        "name":{
            "type":"text",  //字段类型 可以分词
            "index" :true  // 字段可以索引
        },
        "sex":{
            "type":"keyword",  // 不能分词，必须全部匹配
            "index" :true  // 字段可以索引
        },
        "tel":{
            "type":"keyword",  // 不能分词，必须全部匹配
            "index" :false  // 字段不可以索引 就是不能被查询
        }
    }
}

###查询
GET  http://192.168.25.128:9200/user/_mapping

###  增加数据
PUT  http://192.168.25.128:9200/user/_create/1001
Content-Type: application/json

{
    "name": "小米",
    "sex": "男",
    "tel": "1111"
}

###  查询数据
GET   http://192.168.25.128:9200/user/_search
Content-Type: application/json

{
    "query":{
        "match":{
            "tel":"1111"
        }
    }
}
```

## 集群

```

### 集群状态查看
GET   http://192.168.25.129:9200/_cat/nodes

```