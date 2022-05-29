title: linux学习-网络
author: Tany
tags:
  - linux
categories:
  - linux
date: 2020-03-21 18:57:00
---
linux 学习

<!-- more -->

## 配置

### 动态ip修改为静态ip

- 一定要先启用网卡开关 ONBOOT=yes

配置文件 /etc/sysconfig/network-script/ifcfg-ens33

- 修改 dhcp 为 static
- 添加配置项  IPADDR = ip
- 添加网关  GATEWAY = ip
- 添加dns1  = dns  （可以指定多个dns,例子 dns1 dns2  dns3）

 重启服务 ： service network restart

### 配置主机名

查看当前主机名：`hosename`

1. 配置文件 /etc/hostname   (需要重启)
   
   - 直接写入名字
2. hostnamectl set-hostname  xxx （即时生效）

### 配置hosts映射关系

配置文件  /etc/hosts

    - 直接填入`ip  hostname`

### 配置完成后如果无法解析域名，需要修改dns

- 修改NetworkManager.conf 配置文件

   ```
  vi  /etc/NetworkManager/NetworkManager.conf
  #在[main]中添加
  dns=no
   ```

- 修改resolv.conf配置文件
  
  ```
  vim /etc/resolv.conf 
  #添加dns 
  nameserver 114.114.114.114
  ```
- 重启 `systemctl restart NetworkManager`