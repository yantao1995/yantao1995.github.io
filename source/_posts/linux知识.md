title: linux学习-基本
author: Tany
tags:
  - linux
categories:
  - linux
date: 2020-03-20 18:57:00
---
linux 学习

<!-- more -->

## $开头的一些常用特殊指令
```
$#：传入的参数数量
$@：传入的参数列表
$0,$1,...：传入的第 i 个参数
!$ 或者 $_：上一个命令的最后一个参数
$?：上一条命令返回的状态码
$!：Shell 最后运行的后台 Process 的 PID
```

## 软件包管理

#### rpm 红帽软件包管理工具，类似于windows的setup.exe

缺点就是各种依赖关系不好处理，a依赖b，b依赖c这种

- 操作` rpm [-qa 查询安装的所有软件包，-e卸载，-i 安装(-v 显示详细信息，-h显示进度条) --nodeps 不检查依赖强制操作 ]`

#### yum 基于rpm管理，更好用，一键操作

 自动处理依赖关系

- 操作  yum [-y所有都yes，install 安装，update 更新，check-update 检测是否更新，remove删除，list显示信息，clean清理过期缓存，deplist显示所有依赖]
- 修改yun网络源 文件`/etc/yum.repos.d 目录下  CentOs-Base.repo文件`

## 常用

### 服务管理

.target 一组服务的集合
.service 单个服务

##### centos6

service  服务名  [restart | stop | start | status]
所在目录  /etc/init.d/

##### centos7 (也兼容 centos6 的方式)

systemctl  [start |stop | restart | status|disable]  服务名
所在目录  /usr/lib/systemd

### 启动级别

- 3  multi-user.target   多用户控制台界面
- 5  graphical.target  多用户图形界面

查看默认运行级别  `systemctl  get-default`
设置默认运行级别  `systemctl  set-default 运行级别`

切换运行级别  `init 3/5`



