title: 没事干瞎写系列之x的n次方
author: Tany
tags:
  - go
categories:
  - golang
date: 2019-11-25 21:28:00
---
在网上突然看到别人再问2的10次方是几位数，然后就瞎写了一下，在此做笔记记录一下。用的是最蠢的循环

<!-- more -->

######  比如2的100次方，-2的100次方。
- 因为计算机的存储数的位数有限制，所以下面采用了[]int切片来存储每一位数字。
- tips:计算机中，比如想算2^10000，计算机会先算2^5000，再算一次平方，即两个数的乘法。而为了计算2^5000，计算机会先算2^2500再算一次平方。这个算法叫快速幂算法。
- 但是这儿采用了最蠢的循环。
- a: -10<a<10 的整数。  b:  b>=0 整数

```
//IndexMult a的b次方,b为非负整数
func IndexMult(a, b int) ([]int, string) {
    result := make([]int, 1)
    symbol := "+"
    if a < 0 { //正负
        if b%2 != 0 {
            symbol = "-"
            a = 0 - a
        }
    }
    result[0] = a
    if b == 0 {
        result[0] = 1
    }
    for j := 0; j < b-1; j++ {
        nextFlag := 0
        for i := 0; i < len(result); i++ {
            r := a*result[i] + nextFlag
            nextFlag = r / 10
            low := r % 10
            result[i] = low
        }
        if nextFlag != 0 {
            result = append(result, nextFlag)
        }
    }
    return result, symbol
}
```
测试一波(输出顺序从尾巴到头部):
```
func main(){
	mult, symbol := IndexMult(2, 10000)
    fmt.Printf("%s", symbol)
    for i := len(mult) - 1; i >= 0; i-- {
        fmt.Printf("%d", mult[i])
    }
} 
```
- 输出时。从切片的尾部向头部输出就是正确的数字序列。因为数字的最高位是存储在切片的尾部的。