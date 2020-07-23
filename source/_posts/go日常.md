title: go踩坑日记
author: Tany
tags:
  - go
  - go日常
categories:
  - golang
date: 2019-10-24 22:38:00
---
golang日常遇到的一些坑或者小技能实录

<!-- more -->

##### defer链式函数的传递调用
```
type Slice []int
func NewSlice() Slice {
	return make(Slice, 0)
}
func (s *Slice) Add(elem int) *Slice {
	*s = append(*s, elem)
	fmt.Print(elem)
	return s
}
func main() {
	s := NewSlice()
	defer s.Add(1).Add(2).Add(4)
	s.Add(3)
}
```
``` 
输出 1234
```
- 本以为输出3124，或者3421。结果输出1234。
- 原因：defer只能执行一个函数。
- 在defer后是一个链式函数而且defer是一个压栈的输入，4被defer执行，1和2直接被执行，所以打印1234。

##### golang知识体系
- [点击查看这张图](https://raw.githubusercontent.com/gocn/knowledge/master/Go知识图谱.png)

##### 内建函数new与make
- 二者都是用于内存的分配（堆上），但是make只用于slice、map以及channel的初始化（非零值）；而new用于类型的内存分配，并且内存置为零。make返回的还是这三个引用类型本身；而new返回的是指向类型的指针。在使用slice、map以及channel的时候，还是要使用make进行初始化，然后才才可以对他们进行操作。

##### 对切片进行遍历赋值
- 如果不是append直接追加赋值，用索引下标方式赋值一定要先初始化分配空间
```
 a := []int{1,2,3}
var b []struc = make([]struc, len(a))
    for k, _ := range a {
        b[k].Name = "123"
        fmt.Println(b[k])
    }
type struc struct {
    Name string
}
```

##### 协程并发执行等待
```
var wg sync.WaitGroup
for i:=0:i<=10;i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
 //。。。此处执行任务代码
        }()
    }
wg.Wait()
```
- `wg.Wait()`会等待所有的协程结束，才会继续执行下去。其中 `wg.Add(number)`number可以为2，3等等其他值。取决于下面的协程数量，但是每`Add()`开启一个协程，必须对应一个 `Done()`关闭。

##### 当天0点的时间戳
```
t := time.Now()
    tm1 := time.Date(t.Year(), t.Month(), t.Day()-1, 0, 0, 0, 0, t.Location())
    tm2 := tm1.AddDate(0, 0, 1)
```
- 切记 `  t.day() -1` ，否则是明天的0点时间戳

##### 常用类型转换
```
int转string            strconv.Itoa()  //int float string 互转用strconv包
[]byte/byte 转string   string([]byte/byte) 
interface{}转 string   interface{}.(string) 

 ```
 
##### 切片元素删除
```
t := []string{"a", "b", "c", "d", "e", "f", "g", "h"}

    for i := 0; i < len(t); i++ {
        if t[i] == "c" || t[i] == "c" || t[i] == "d" {
            // 将删除点前后的元素连接起来
            fmt.Println(i, t[i])
            t = append(t[:i], t[i+1:]...)
            i--
        }
        fmt.Println(i)
    }
    fmt.Println("----------")
    for i := 0; i < len(t); i++ {
        fmt.Println(i, t[i])
    }
```
元素删除后使用i- - 将遍历位置切换到连接点。
否则会出`panic: index of range`。

##### 字符串比较
```
strings.Compare(a, b) 
       
```
strings.Compare() 效率比 == 效率高

##### 内层结构体嵌套
```
func main(){
    var s Student
    s.Stu = make([]Stu2, 0) //初始化
    str, _ := json.Marshal(&s)
    fmt.Println(string(str))
}

type Student struct {
    Name string
    ID   int
    Stu  []Stu2
}

type Stu2 struct {
    N2 int
    N3 int
}
```
- 在将结构体内嵌套的切片结构体转成json数据时，如果不用make初始化:
`s.Stu = make([]Stu2, 0)`转换后的json会变成`{"Name":"","ID":0,"Stu":null}`
- 使用make初始化之后json为： `{"Name":"","ID":0,"Stu":[]}`

##### windows下使用阿里云代理包下载
```
$env:GO111MODULE="on"
$env:GOPROXY="https://mirrors.aliyun.com/goproxy/"
```

##### Inf 阶码溢出
- inf前面带+-符号的，代码高位溢出。小数点位后面无限大。
- 解决方法：使用fmt.Sprintf("%0.2f",浮点数) 输出一个字符串的固定位的浮点数，在把这个字符串转为float。

##### nohup文件过大
- 清空：方法1 `cp /dev/null nohup.out` ，方法2 `cat /dev/null > nohup.out`

##### json操作
- 因为原生包上操作json需要使用struct来映射到对应的属性上去，所以可以利用两个包来直接操作json的[]byte数组
- set的相关操作：`https://github.com/tidwall/sjson`
- get的相关操作：`https://github.com/json-iterator/go` 滴滴平台的包