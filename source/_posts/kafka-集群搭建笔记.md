title: kafka 集群搭建笔记
author: Tany
tags:
  - kafka
categories:
  - kafka
date: 2021-12-28 16:09:00
---
学习 kafka 的简单笔记，记不住了就来看看。

<!-- more -->


## 准备工作

- 准备并配置 jdk 环境

- - -  【以下步骤均在在3台机器上操作】

- 启动zookeeper 集群 【高于2.8版本就不需要启动了】

- 解压 kafka 

```
 tar  -zxvf kafka_2.13-2.6.3.tgz
```

- 进入 config 目录修改配置

```
vim  server.properties

//修改当前机器的 broker id ，分别填写3台机器的编号
broker.id=0

// 此log 非彼 log， 这是kafka的消息数据文件
log.dirs=/opt/moudle/kafka_2.13-2.6.3/kafkalogs

// 修改 zookeeper 的集群地址和端口
zookeeper.connect=192.168.25.129:2181,192.168.25.130:2181,192.168.25.131:2181

//修改本机的监听端口
listeners=PLAINTEXT://192.168.25.129:9092

```

## 启动&停止

- 进入 kafka 根目录

- 非后台启动

```
bin/kafka-server-start.sh config/server.properties

```
- 后台启动

```
bin/kafka-server-start.sh -daemon config/server.properties

```


- 安装jps后，使用命令  `jps` 即可看到启动后的程序

```
jps

---
2208 QuorumPeerMain
3811 Kafka
3885 Jps
---
```

- 停止

```
bin/kafka-server-stop.sh

```

## 简单测试

-  topic

```
//创建  
//partitions 尽量与broker数量相同，读写性能高，replication-factor 不能超过broker数量

bin/kafka-topics.sh --zookeeper node1:2181 --create --topic test2 --partitions 3 --replication-factor 3

---
Created topic test2.
---

//查看

bin/kafka-topics.sh --zookeeper node1:2181 --describe --topic test1

---

Topic: test1    PartitionCount: 3       ReplicationFactor: 3    Configs:
        Topic: test1    Partition: 0    Leader: 2       Replicas: 2,0,1 Isr: 2,1,0
        Topic: test1    Partition: 1    Leader: 2       Replicas: 0,1,2 Isr: 2,0,1
        Topic: test1    Partition: 2    Leader: 2       Replicas: 1,2,0 Isr: 2,1,0
---


```

- console-producer

```
//向 test1 发布消息
bin/kafka-console-producer.sh  --bootstrap-server node1:9092 --topic test1

```

- console-coumser （对于老的消费者，由--zookeeper参数设置；对于新的消费者，由--bootstrap-server参数设置如果使用了--zookeeper参数,那么consumer的信息将会存放在zk之中）

```
bin/kafka-console-consumer.sh --bootstrap-server node2:9092 --topic test1 --from-beginning 
```