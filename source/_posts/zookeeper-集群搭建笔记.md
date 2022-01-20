title: zookeeper 集群搭建笔记
author: Tany
tags:
  - zookeeper
categories:
  - zookeeper
date: 2021-12-20 15:42:00
---
学习 zookeeper 的简单笔记，记不住了就来看看。

<!-- more -->


## 配置

- 准备3台机器，然后分别进行下面的操作

- 安装并配置jdk环境

- 解压zookeeper

```
tar -zxvf apache-zookeeper-3.5.9-bin.tar.gz
```

- 进入conf目录

```
cp zoo_sample.cfg zoo.cfg
vim zoo.cfg

//其他配置默认

//修改 dataDir数据文件存放地址和集群
dataDir=/opt/moudle/apache-zookeeper-3.5.9-bin/data

//增加 dataDir数据文件存放地址和集群
//说明 节点编号 =  ip ：zookeeper服务端口 ：节点选举端口 
server.0=192.168.25.129:2888:3888
server.1=192.168.25.130:2888:3888
server.2=192.168.25.131:2888:3888

```

- 进入刚刚配置的 dataDir 文件夹

```
//创建 myid 文件
vim myid

//写入id,比如当前为 server.0 节点，就直接写入0
0
```
## 异常

- 若启动失败，检查防火墙。

```
//查看防火墙状态
service iptables status
//关闭防火墙
chkconfig iptables off

```

## 启动

- 进入 bin目录

```
//启动
zkServer.sh start
//停止
zkServer.sh stop
// 重启
zkServer.sh restart
// 状态
zkServer.sh status
```

