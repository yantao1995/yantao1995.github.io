title: java相关的技术经验汇总
categories:
  - java
tags:
  - java
  - spring
  - 并发
author: Tany
date: 2019-08-13 09:03:00
---
java的一些基础知识汇总

<!-- more -->

##### SpringMVC   
https://blog.csdn.net/joker_honey/article/details/80920730

##### 单例 
  single  构造函数私有化，一个公有的函数获取和创建...。或者使用枚举 Enum

##### 反射 ：得到元数据的行为。Class
`getPackage();getSuperClass();getMethod();......`

##### 序列化:
`ObjectInputStream ; ObjectOutputStream  `

##### IoC:控制反转
使用反射机制并结合属性配置文件的工厂模式。去构建依赖对象(Factory生产。任意多子类，工厂类不需要修改)由Ioc容器来控制对象的创建及注入依赖对象

##### AOP：切面：多个模块拥有同一种功能。(横切)
动态代理 ：（静态代理：《使用了聚合：让代理类持有一个委托类的引用》一个类中有另一个类的对象。（也可以继承实现，但是继承很不灵活））
	`public static Object newProxyInstance(ClassLoader loader, Class<?>[]  interfaces,InvocationHandler h) throws IllegalArgumentException`
参数1：类加载器，指明哪个类的被代理。
参数2：需要实现的《接口类》的Class数组（因为是数组，所有可以有多个）
参数3：调用处理方法。调用对象时，做什么。函数的参数。
   
##### 动态代理详细剖析： 
 http://developer.51cto.com/art/201606/512434.htm 

##### 消息 
 http://www.importnew.com/21875.html


##### 分布式并发  
http://ifeve.com/java-concurrency-thread-directory/
######  分布式集群高并发详细：
https://blog.csdn.net/u011277123/article/details/54015614


##### 多线程	 
继承Thread类 或者 实现 Runnable接口。 Runnable可以实现数据共享

##### 队列  
Queue接口与List、Set同一级别，都是继承了Collection接口。


##### NIO 与 IO：
IO： 面向流 单向的 面向缓冲区：通道可以是单双向的  阻塞IO
NIO：面向缓冲区(Buffer Oriented)：通道可以是单向的，也可以是双向的非阻塞IO(Non Blocking IO)缓冲区(存储)和管道(数据源和结点的连接传输Buffer)

##### PrintWriter 
文本文件  处理字符流，一次写入2个字节，
##### PrintStream  
处理字节流，一次写入一个字节