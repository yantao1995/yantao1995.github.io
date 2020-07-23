title: go + mysql
categories:
  - golang
tags:
  - golang
  - orm
  - mysql
author: Tany
date: 2019-09-12 15:02:00
---
go对mysql导入，增删改查。

<!-- more -->

#### 高级编程 
https://chai2010.gitbooks.io/advanced-go-programming-book/content/

##### 下载驱动包：
	$ go get github.com/go-sql-driver/mysql

###### （1）sql.Open("mysql", "username:pwd@/databasename")

功能：返回一个DB对象，DB对象对于多个goroutines并发使用是安全的，DB对象内部封装了连接池。

实现：open函数并没有创建连接，它只是验证参数是否合法。然后开启一个单独goroutines去监听是否需要建立新的连接，当有请求建立新连接时就创建新连接。

注意：open函数应该被调用一次，通常是没必要close的。

 

###### （2）DB.Exec()

功能：执行不返回行（row）的查询，比如INSERT，UPDATE，DELETE

实现：DB交给内部的exec方法负责查询。exec会首先调用DB内部的conn方法从连接池里面获得一个连接。然后检查内部的driver.Conn实现了Execer接口没有，如果实现了该接口，会调用Execer接口的Exec方法执行查询；否则调用Conn接口的Prepare方法负责查询。

 

###### （3）DB.Query()

功能：用于检索（retrieval），比如SELECT

实现：DB交给内部的query方法负责查询。query首先调用DB内部的conn方法从连接池里面获得一个连接，然后调用内部的queryConn方法负责查询。

 

###### （4）DB.QueryRow()

功能：用于返回单行的查询

实现：转交给DB.Query()查询

 

###### （5）db.Prepare()

功能：返回一个Stmt。Stmt对象可以执行Exec,Query,QueryRow等操作。

实现：DB交给内部的prepare方法负责查询。prepare首先调用DB内部的conn方法从连接池里面获得一个连接，然后调用driverConn的prepareLocked方法负责查询。

Stmt相关方法：

```
st.Exec()
st.Query()
st.QueryRow()
st.Close()
```

 

###### （6）db.Begin()

功能：开启事务，返回Tx对象。调用该方法后，这个TX就和指定的连接绑定在一起了。一旦事务提交或者回滚，该事务绑定的连接就还给DB的连接池。

实现：DB交给内部的begin方法负责处理。begin首先调用DB内部的conn方法从连接池里面获得一个连接，然后调用Conn接口的Begin方法获得一个TX。

TX相关方法：

//内部执行流程和上面那些差不多，只是没有先去获取连接的一步，因为这些操作是和TX关联的，Tx建立的时候就和一个连接绑定了，所以这些操作内部共用一个TX内部的连接。
```
tx.Exec() 
tx.Query()
tx.QueryRow()
tx.Prepare()
tx.Commit()
tx.Rollback()
```
tx.Stmt()//用于将一个已存在的statement和tx绑定在一起。一个statement可以不和tx关联，比如db.Prepare()返回的statement就没有和TX关联。