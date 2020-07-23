title: go协程并发channel通信
author: Tany
tags:
  - go
  - channel
categories:
  - golang
date: 2019-11-20 20:22:00
---
在go语言中文网上看到一个面经，其中的一道题

<!-- more -->

######  协程间的通信
- 启动两个协程, 一个输出 1,3,5,7…99, 另一个输出 2,4,6,8…100 最后 STDOUT 中按序输出 1,2,3,4,5…100
- 因为在main函数中，为了让main函数等待协程执行，所以使用了sync.WaitGroup来等待协程执行完成
- 最终优化后能正确输出的代码：
```
    var wg sync.WaitGroup
    ch := make(chan int)  
    ch2 := make(chan int) 
    wg.Add(2)
    go func() {
        for i := 1; i <= 50; i++ {
            fmt.Printf("%d\t", <-ch)
            ch2 <- 2 * i 
        }
        wg.Done()
    }()
    go func() {
        for j := 1; j <= 50; j++ {
            ch <- 2*j - 1
            fmt.Printf("%d\t", <-ch2) 
        }
        wg.Done()
    }()
    wg.Wait()
```
输出：
```
1       2       3       4       5       6       7       8       9       10      11      12      13      14      15      16      17      18      19      20      21      22      23      24      25        26      27      28      29      30      31      32      33      34      35      36      37      38      39      40      41      42      43      44      45      46      47      48      49        50      51      52      53      54      55      56      57      58      59      60      61      62      63      64      65      66      67      68      69      70      71      72      73        74      75      76      77      78      79      80      81      82      83      84      85      86      87      88      89      90      91      92      93      94      95      96      97        98      99      100
```
——————————————————————————————————————
###### 原始的错误代码
- 最开始写的代码是这样的：
```
    var wg sync.WaitGroup
    ch := make(chan int)
    wg.Add(2)
    go func() {
        for i := 1; i <= 50; i++ {
            fmt.Printf("%d\t", <-ch)
        }
        wg.Done()
    }()
    go func() {
        for j := 1; j <= 100; j++ {
            ch <- j
            j++
            fmt.Printf("%d\t", j)
        }
        wg.Done()
    }()
    wg.Wait()
```
执行后的结果：
```
2       1       3       4       6       5       7       8       10      9       11      12      14      13      15      16      18      17      19      20      22      21      23      24      26        25      27      28      30      29      31      32      34      33      35      36      38      37      39      40      42      41      43      44      46      45      47      48      50        49      51      52      54      53      55      56      58      57      59      60      62      61      63      64      66      65      67      68      70      69      71      72      74        73      75      76      78      77      79      80      82      81      83      84      86      85      87      88      90      89      91      92      94      93      95      96      98        97      99      100
```
分析：由于使用的是无缓冲通道，当协程并发执行时,在第二个`go func(){}` 中`ch <- j`将j的值传入通道后,第一个`go func(){}`会进入非阻塞状态，此时第二个`go func(){}`直到下次给通道赋值前。两个协程都处于无限制的并发状态，导致了出现的2 1 3 4 6 5的输出可能性。

###### 第一次优化 
- 在第二个协程中的`time.Sleep(10)`加入等待时间。让第二个协程在通知第一个协程之后，保证第一个协程的`fmt.Printf("%d\t", <-ch)`能先打印出信息。当然缺点很明显。这样就平白无故的浪费了  10 x 50 毫秒的时间。

```
    var wg sync.WaitGroup
    ch := make(chan int)
    wg.Add(2)
    go func() {
        for i := 1; i <= 50; i++ {
            fmt.Printf("%d\t", <-ch)
        }
        wg.Done()
    }()
    go func() {
        for j := 1; j <= 100; j++ {
            ch <- j
            time.Sleep(10)
            j++
            fmt.Printf("%d\t", j)
        }
        wg.Done()
    }()
    wg.Wait()
```
输出：1 -> 100 正常输出
###### 第二次优化 
- 去掉了`time.Sleep()`在第一个协程和第二个协程中也使用channel通道来传递信息，保证两边按顺序输出。
```
    var wg sync.WaitGroup
    ch := make(chan int)  
    ch2 := make(chan int) 
    wg.Add(2)
    go func() {
        for i := 1; i <= 50; i++ {
            fmt.Printf("%d\t", <-ch)
            ch2 <- 1 //通知另一个 协程 处理完成
        }
        wg.Done()
    }()
    go func() {
        for j := 1; j <= 100; j++ {
            ch <- j
            j++
            <-ch2 //等待另一个协程处理完成
            fmt.Printf("%d\t", j)
        }
        wg.Done()
    }()
    wg.Wait()
```
输出：1 -> 100 正常输出

###### 最后优化了变量的使用，代码更清晰。第一步操作