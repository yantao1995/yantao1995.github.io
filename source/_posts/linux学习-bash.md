title: linux学习-bash
author: Tany
tags:
  - linux
categories:
  - linux
date: 2020-03-24 18:57:00
---
linux 学习

<!-- more -->


## 扩展

### 在线帮助文档

#### man

 `man  [命令的名称]`
type [命令] 判断命令类型 ,内部命令需要 `man -f [内部命令]`

- 内部命令 cd ,exit
- 外部命令 ps,ls .....

#### help

help [内部命令]
只能查看内部命令

#### --help

[外部命令] --help
只能查看外部命令

## 常用

### 普通

`pwd,ls,cd,mkdir,rmdir,touch,cp,rm,mv,catmore,less,echo,alias,head,tail,history,date,sort`

- 输入重定向和追加:  `>` 覆盖  , `>>` 追加
- ln : 仅删除软连接 `rm [名] `   , 删除连接文件夹内文件 `rm  [名]/  ， [-s 不加就是创建硬链接，底层会复制一份inode信息]`
### 常用易忘命令参数
#### sort
  ```
  -k [n]  //按第n列升序
  -r  //降序
  -n //按数值大小排序
  -t [分隔字符]  //指定分隔字符
  ```

### 管理

#### 账户

##### 用户

创建用户 `useradd [-g [组名]] [用户名]`
更改密码 `passwd [用户名]`
查看用户是否存在  `id [用户名]`
切换用户 `su [用户名]`
当前用户  最原始的会话 `who am i` , 目前的会话 `whoami`
赋予root权限执行命令  `sudo  [命令]`

- 需要先将用户权限添加至 `/etc/sudoers`
- 用户权限  `用户名  	ALL=(ALL)   ALL`
- 组权限  `%组名   ALL=(ALL)   ALL   [NOPASSWD:all  （可选项，用于跳过输密码） ]`

##### 组 ： 集中化管理用户

用户删除  `userdel  [-r:是否删个人目录] [用户名]`
用户组  配置文件 `/etc/group`
新建组 `groupadd [名]`
修改用户所在组 `usermod -g  [组名]  [用户名]`
修改组的组名 `groupmod  -n [新组名]  [旧组名]`
删除组 `groupdel [名]`

#### 权限

- chmod 修改文件
  查看  `ll  或者  ls -l        - 普通文件 d 文件夹  l 链接文件`
  读写运行  ` rwx` ,   所属用户 | 组用户  | 其他用户
  
  - 删除文件，需要对文件父目录有w写权限
  - 进入目录，需要有对该文件的x运行权限
  - 修改权限 `chmod [-R (针对目录下的所有文件)] [ugoa (用户/组/其他用户/全部)]  [+-=]  [rwx]  文件/目录`
- chown 改变所有者 `chown [-R (针对目录向下递归)]  [目标用户]  [文件/目录]`
- chgrp 改变所属组  `chgrp [目标组] [文件/目录]`

#### 搜索

- find 找文件/目录  ` find  [ （路径匹配）  (-name 名字) (-user 所属用户) （-size 文件大小 [ +2M（大于2M）]）] [文件名]`
- locate 速度快，在文件系统数据库内找 `locate [名]`，每天更新，也可以手动更新数据库 `updatedb`
- which 查找命令位置,  `which [命令]`
- grep 过滤查找  `grep [-n 显示行号] [关键字]  [文件名]`
- wc  统计词数   `wc [文件名]`  结果 `x1  x2  x3` 分别为  行数，单词数，字节数

#### 压缩解压

- gz压缩解压 `gzip/gunzip [文件名] ` 不保留源文件，只能压缩文件，不能压缩目录
- z压缩解压 `zip [-r 压缩目录]     unzip [-d 存放目录]`   要保留源文件，可以压缩目录
- tar 打包归档  `tar [-c 产生.tar文件, -v 显示详细信息, -f 指定压缩后的文件名，-z 打包同时压缩(使用的gzip)，-x解包.tar文件,-C指定解压目录] `
  - 压缩 `tar -zcvf  [文件名 ]  [....文件]`
  - 解压 `tar -zxvf  [文件名 ] [-C]  [指定目录]`

#### 磁盘

- tree 显示目录树 `安装yum install tree`
- 显示当前文件本身的大小 `ls -lh `
- du查看当前目录占用所有的大小  `du [-h显示单位大小,-a不仅查看子目录，还有文件，-c结尾额外显示总和，-s只显示总和，--max-depth=n指定统计子目录深度的第n层] [目录/文件]`
- df 查看磁盘剩余空间  `df [-h 显示单位大小]`
- mount/umount 挂载/卸载  `mount  [-t 文件系统类型，-rw 读写方式，-ro只读] [设备位置]  [挂载点] `
  - 设置开机启动自动挂载，修改文件  `/etc/fstab`  里面新增一行挂载内容
- fdisk分区  `fdisk [-l 查看分区详情，-n添加一个分区]`
- lsblk 查看设备挂载情况  `lsblk [-f 查看设备详细情况，文件情况]`

#### 内存

- free  查看内存使用情况  `free [-h 显示单位大小]`

#### 系统管理

- ps 查看进程 只显示当前终端用户的 `ps`  参数带 - 的表示标志unix风格，不带-的是BSD风格 一般想看资源占用率 `ps aux`  想看进程父子关系 `ps -ef`
  
  - `ps [a带终端所有用户进程，x当前用户的所有进程，u面向用户友好的显示风格]` 多显示了内存和cpu使用
  - `ps[-e所有进程,-u某个用户关联的进程,-f完整格式的进程列表]` 多显示了父进程ppid
- kill 终止进程   【慎用 killall 终止所有与该进程有关的进程】`kill  [-(1-62)信号值【一般-9 SIGKILL】，-l查看信号值列表] [进程id]`
- pstree 进程树  `ps [-p 查看pid ，-u 进程所属]`
- top 实时查看进程状态  `top [-d秒数 指定每隔几秒更新，-i不显示闲置或僵死进程，-p指定进程id来监控某个进程]` 功能 `u 筛选user，k终止进程`
- netstat 显示网络状态  `netstat [-a 显示所有list和socket，-n不显示别名能显示数字的全部显示数字，-l仅列出监听的服务状态，-p显示哪个进程在调用]` 常用查看进程网络信息 `netstat -anp`  常用查看网络端口 `netstat -nlp`
- crontab 系统定时任务,开启守护进程 `systemctl start crond` 命令行 `crontab [-e 编辑任务，-l查询任务， -r删除当前用户的所有任务]`  编辑 ` * * * * * 分别表示 分钟，小时，天，月，星期几`