title: go slice
categories:
  - golang
tags:
  - go
  - slice
  - 切片
date: 2019-09-20 09:38:00
---
切片的结构，创建，扩容

<!-- more -->

##### 切片结构
```
type slice struct {  
    array unsafe.Pointer
    len   int
    cap   int
}
```
Pointer：指向一个数组的指针

len： 代表当前切片的长度

cap ：是当前切片的容量，总是大于等于 len 。

##### 切片创建

```
//1.类型自动推导
s:=[]int{1,2,3,4}
fmt.Println(s, len(s), cap(s))
```
```
//2.借助make函数，格式为：make(切片类型，切片长度，切片容量)
s1:=make([]int,5,10)
fmt.Println(s1, len(s1), cap(s1))
```
```
//3.借助make函数，格式为：make(切片容量，切片长度)
s2:=make([]int,5)
fmt.Println(s2, len(s2), cap(s2))
//这种情况切片容量与切片长度相等
```
```
//4.通过现成的数组（或者切片）进行创建
a:=[5]int{1,2,3,4,5}
s3:=a[1:3:5]
fmt.Println(s3, len(s3), cap(s3)) 

```
##### 切片扩容 

[扩容出处，简书博主： l_sivan](https://www.jianshu.com/p/303daad705a3)
1. append单个元素，或者append少量的多个元素，这里的少量指double之后的容量能容纳，这样就会走以下扩容流程，不足1024，双倍扩容，超过1024的，1.25倍扩容。
2. 若是append多个元素，且double后的容量不能容纳，直接使用预估的容量。
3. 以上两个方法得到新容量后，均需要根据slice的类型size，算出新的容量所需的内存情况capmem，然后再进行capmem向上取整，得到新的所需内存，除上类型size，得到真正的最终容量,作为新的slice的容量。