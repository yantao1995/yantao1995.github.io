title: clair镜像扫描
author: Tany
tags:
  - k8s
  - clair
  - 镜像扫描
  - docker
categories:
  - k8s
date: 2019-08-19 18:00:00
---
clair简介，docker镜像简介，准备工作

<!-- more -->

#### cliar简介
Clair是一个开源工具，用于扫描Docker镜像的漏洞。

Clair没有Web端，也没有命令行工具。只能通过REST API或第三方CLI工具。

 获取clair镜像：`docker pull quay.io/coreos/clair`

#### 镜像简介
docker镜像由 1+n层组成，每一层都是tar文件块存储在docker注册表里。

#### 需要的工具

1. clair
  1. REST API服务
  2. CVE Updater 更新漏洞数据库
  3. CVE 数据源列表
2. postgreSQL 数据库
  1. 存储漏洞信息
  2. 上传docker镜像分析结果

cliar：[api_v1官方文档](https://coreos.com/clair/docs/latest/api_v1.html)

#### 接口
1. POST/layers    将docker镜像层推送到Clair进行分析
2. GET/layers/:name   获取已发现漏洞的Docker镜像层的信息

#### k8s集群内配置文件
1. [clair_config/config.yaml](https://raw.githubusercontent.com/coreos/clair/master/config.yaml.sample) 
2. docker-compose.yaml (docker-compose up/down)

参考：[Static Analysis of Docker image vulnerabilities with Clair](https://www.nearform.com/blog/static-analysis-of-docker-image-vulnerabilities-with-clair/)