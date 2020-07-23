title: gorm及gin入门及踩坑日记
author: Tany
tags:
  - go
  - gorm
  - gin
categories:
  - golang
date: 2019-10-21 22:50:00
---
golang的关系型数据库框架gorm 和 web框架gin的一些踩坑实录 

<!-- more -->

###### mysql值映射
```
enmu类型 对应 string
int 类型在mysql中的int长度为最大16位。

```

###### update更新
- `where()` 函数应放在 `update()`函数之前,否则无法完成where的字句查询 。
- 在`Update(column,value)` 中value与column的类型需要相对应。非对应类型：比如本应填整型的地方，如果填写了"2"，则会自动映射成ASCII值50。

###### gorm结构体映射及json构造映射
 ```
 type GormAndJSON struct{
		UID     int 	  `gorm:"cloumn:uid" json:"uid"`
		Date    time.Time `gorm:"column:date" json:"date"`
 }
 ```
 - 结构体字段名 UID 和 Date 首字母一定要大写。大写类似java的public，小写类似private。小写无法完成gorm的cloumn字段映射，也不能json.Marshal，json.Unmarshal映射
 - mysql数据库的 datetime 字段，在结构体中定义类型为 time.Time 的可以接收。
 - gorm映射时`gorm:"column:date"` 不加column有时会导致某些字段无法正确映射，int类型字段可以映射，但string，datetime等字段无法完成映射。
 
###### gin接收 GET 请求的参数
 ```
  r := gin.Default()
    r.GET("/ping", func(c *gin.Context) {
       param1 := c.GetString("param1")
       param2 := c.Query("param2")
    })
    ```
- 此时的param1为空值 ""
- 此时的param2能正常获取到参数

###### gin接收POST请求参数
```
c :=  *gin.Context
name := c.PostForm("name")
message := c.GetString("message")
```
- 此时的name能正常获取到参数
- 此时的message为空值""

###### gin设置头部 返回内容为json
```
c := *gin.Context
c.Set("content-type", "application/json")
无效时使用 c.Header("content-type", "application/json")
```
###### gorm 更新表的坑
```
// 警告:当使用struct更新时，FORM将仅更新具有非空值的字段
// 对于下面的更新，什么都不会更新为""，0，false是其类型的空白值
db.Model(&user).Updates(User{Name: "", Age: 0, Actived: false})
```
###### gin文件下载
```
c.File(filepath+filename)
```
###### gorm 多库联合查询
指定库名，比如 `demo.test_t1` 表名前加库名

######  gin限制ip登录
```
ipList := []string{
        "127.0.0.1",
        "localhost",
    }
    flag := false
    clientIP := c.ClientIP()
    for _, host := range ipList {
        if clientIP == host {
            flag = true
        }
    }
    if !flag {
        c.String(404, "拒绝访问")
        return
    }
```
###### 跨库查询是否有特定的行数据存在
```
rows, _ := db.Table(" lxg.lxg_shoping_order as lso ").Where(" lso.`status` = '5' and lso.id = ? ", "2").Rows()
    if rows.Next() {
        fmt.Println("true")
    } else {
        fmt.Println("false")
    }
```
- true为存在至少一行，false表示不存在