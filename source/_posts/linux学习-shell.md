title: linux学习-shell
author: Tany
tags:
  - linux
categories:
  - linux
date: 2020-03-26 18:57:00
---
linux 学习

<!-- more -->

### 入门

#### 格式

第一行指定解释器,比如指定bash解释器 `#!/bin/bash`

#### 运行

- shell解释器执行【子shell内执行】 `bash  [脚本名]`   `sh  [脚本名]`
- 相对路径或者绝对路径运行【子shell内执行】 【需要x权限】 ` /root/xxx/xx/[脚本名]` `./[脚本名]`
- 不打开子shell启动【当前shell内执行，作用域为当前shell】 `source [脚本名]`  `.  [脚本名]`
- 不能直接 `[脚本名]` 启动,因为没有加入到系统环境变量，直接写命令回去环境变量里面找

### 变量

#### 系统预定义变量

$HOME,$PWD,$SHELL,$USER 等

#### 自定义变量

- 定义 ` 变量名=变量值`   注意，=号前面不能有空格
- 默认是字符串类型

#### 变量作用域

- 子shell变量导出到父shell `export [变量名]` 不用加$
- 如果当前子shell更改了全局变量，只能在当前的shell生效

#### 操作

- 只读变量 `readonly  [变量名]=[值]` 不能被unset
- 撤销变量  `unset [变量名]`

#### 特殊变量

- 脚本传参,$n表示第n个参数，0表示脚本名称，1-9表示的参数，10以上的需要大括号包住, `\{10\}`，脚本内使用的是双引号,例` echo "hello $1" ` ,单引号则原样输出字符串
- 获取参数个数 `$#`
- 所有参数 ，整体，【用于看所有参数 `】$*`
- 所有参数，数组 ，【用于for循环遍历】`$@`
- 最后一次执行命令的返回状态 成功0，失败不为0 `$?`
- 【注意】 `$* 和 $@` 用于 `for 循环中`, 需要用 `""`包裹才能生效
    例如` for a in "$*" `
### 运算符表达式

【 注意 】`(())` 里面可以直接用数学表达式
【注意】 `[ ]` 里面必须是 `-lt ,-gt` 这种表达式
【注意】 表示序列 {} ,示例: `{1..100}` 表示1到100的序列

- 写法1：执行运算【就是1和+和2当参数传给expr】 `expr 1 + 2 `  1+2 中间有空格，*有点特殊，需要\来做转义
- 写法2：简化上面的  `$((运算式)) ` 或者 `$[运算式]` 不需要转义了)
- 命令替换 【获取运算式的结果赋值】 `a=$（expr 5 \* 2）` 或者使用 反引号  `` `  expr 5 \* 4 ` ``

### 条件判断

- if 判断
  
  - 单分支
    写法1：
    
    ```
    if [ 条件判断 ] ; then
    	 程序
    fi
    ```
    
    写法2：
    
    ```
    if [ 条件判断 ] 
    then
    	 程序
    fi
    ```
    
    写法3: 输入到一行
    
    ```
    a=15
    if [ $a -gt 10 ];then echo ok ;fi
    ```
    
    注意点：如果存在脚本需要参数
    
    ```
    #!/bin/bash
    if [ $1 = xiaoming ]
    then
    echo "welcome $1"
    fi
    ```
    
    在不传参数的情况下，会报错 `./if.sh: line 3: [: =: unary operator expected`
    可以优化脚本为：
    
    ```
    #!/bin/bash
    if [ "$1"x = "xiaoming"x ]
    then
    echo "welcome $1"
    fi
    ```
    
    多个条件判断:
    
    - 方式1：
    
    ```
    a=25
    if [ $a -gt 18 ] && [ $a -lt 35 ];then echo OK;fi
    ```
    
    - 方式2： 使用 `-a并且  -o 或者 ` 来连接
    
    ```
    a=25
     if [ $a -gt 18 -a $a -lt 35 ];then echo OK;fi
    ```
  - 多分支
    
    else :
    
    ```
    #!/bin/bash
      if [ $1 -gt 20 ]
      then
      echo "$1 大于 20"
      else
      echo "$1 小于 20"
      fi
    ```
    
    elseif : 写成  elif
    
    ```
    #!/bin/bash
     if [ $1 -gt 20 ]
     then
     echo "$1 大于 20"
     elif [ $1 -eq 20 ]
     then
     echo "$1=20"
     else
    echo "$1 小于 20"
    fi
    ```
- case 语句
  
  - case 行尾必须为单词 `in` ，每个匹配必须有 `)`作为结尾
  - 双分号 `;;` 表示命令序列结束，相当于 break语句
  - 最后的 `*)`表示默认模式，相当于default
    
    示例：
    
    ```
    #!/bin/bash
    case $1 in
    1)
    echo "值为1"
    ;;
    2)
    echo "值为2";
    ;;
    *)
    echo  "进入default分支"
    echo  "可以多行命令"
    ;;
    esac
    ```
- for 循环
  
  - 基本语法1：
    
    ```
    for (( 初始值; 循环控制条件; 变量变化 ))
    do
     程序
     done
    ```
    
    示例:
    
    ```
    #!/bin/bash
    
    for((a=1;a<=10;a++))
    do
       echo "a=$a"
    done
    ```
  - 基本语法2：
    
    ```
    for 变量 in  值1  值2  值3...
    do
    	程序
    done
    ```
    
    示例1：
    
    ```
    #!/bin/bash
    
    for os in linux windows mac
    do
         echo "os=$os"
    done
    ```
    
    示例2：
    
    ```
    #!/bin/bash
    
    for i in {1..100}
    do
         sum=$[ $sum + $i]
    done
    echo $sum
    ```
- while 循环
  语法:
  ```
  while [条件表达式] 可以使用 (()) ,也可以使用 []
  do
    程序
  done
  ```
  示例:
  ```
  #!/bin/bash
  a=1
  while [ $a -lt 10 ]
  do
    echo "a=$a"
    a=$[ $a+1]
  done

  ```
  写法2：【使用let】【+=前面不能有空格】
  ```
  #!/bin/bash   
  a=1
  while [ $a -lt 10 ]
  do
    let sum+=a
    let a++
  done
 
  ```
### read读取控制台输入
  语法 `read [-p指定读取值时的提示符,-t指定读取等待时间(不加一直等待)] [参数(指定读取值的变量名)] `
  示例：
  ```
  #!/bin/bash
  read -t 10 -p "请输入名字:" name
  echo "welcome $name"

  ```
### 函数

#### 系统函数
  命令替换，脚本内调用  `$(命令+参数)`
  例子：函数接收文件名，然后自定义名称加上时间戳
  ```
  #!/bin/bash
  filename="$1"_log_$(date +%s)
  echo $filename
  ```
  - basename : 取文件名  `basename [文件名]  [去掉后缀]` 得到文件名
  - dirname : 获取路径的文件夹路径 `dirname  [路径]` 得到当前文件夹路径
#### 自定义函数
  - 语法：
  ```
  [function]  funname [()]  //function和后面中括号里面的小括号都可以省略
  {  // 这个大括号可以放上面一行
    程序;
    [return int;]  //可以省略
  }
  ```
  - 注意
    1. 调用前必需声明
    2. 函数返回值，只能通过 `$?` 来获得，返回值只能为 `0-255`

  示例1：接收返回值,只能返回0-255的值
  ```
  #!/bin/bash
  function add(){
    s=$[$1+$2]
    return $s;
  }
  read -p "输入第一个参数:" a
  read -p "输入第二个参数:" b
  add $a $b
  echo "和:"$?
  ```
  示例2：命令替换
  ```
  #!/bin/bash
  function add(){
    s=$[$1+$2]
    echo $s
  }
  read -p "输入第一个参数:" a
  read -p "输入第二个参数:" b
  sum=$(add $a $b)
  echo "和:"$sum
  echo "和的平方:"$[$sum * $sum]
  ```
### 正则
  - 字符 ^  匹配一行的开头,例 ` ^a ` ,会匹配所有以a开头的行
  - 字符 \$  匹配一行的结束,例 `t$`,会匹配所有以t结尾的行，如果要对本符号进行匹配，则需要转义 '\\$',必须用单引号
  - 字符 .  匹配一个任意字符,例子 `r..t`,会匹配出 root，ra/t这种
  - 字符 * 上一个字符出现0-n次,例子 `.*`，会匹配任意字符串
  - 字符区间 [] ,表示范围内任意一个字符都可以,例子 `[6,8]`匹配6或者8，例子 `[6-8]`匹配6到8，例子 `[6-8,10-12]`匹配6到8,或者10到12

  - grep使用`-E`参数表示开启支持扩展的正则，可以使用`{n}`表示精确的能匹配到的个数,例如，匹配手机号 `grep ^1[3,4,5,7,8][0-9]{9}$`

### 文本处理
#### cut 文件切割
  从文件中剪切字节，字符和字段并输出，主要分隔列
  使用:`cut [-f指定提取第几列,-d指定列的分隔符,默认制表符\t,-c按字符切割后面跟n表示提取第n列]`
  技巧: `cur -f 6- ` 表示 第6到末尾列
  例子，获取所有ip: `ifconfig | grep netmask | cut -d " " -f 10 ` 因为ip在netmask那一行，所以用来筛选，切割时，前面有空格，所以第10列

#### awk 强大的文本分析，默认以空格切割
##### 基本用法: 
  ```
  awk [-F指定分隔符，-v赋值一个用户定义变量] ‘/pattern1/{action1}  /pattern2/{action2}... ’ filename
  ```
  pattern: 表示 awk在数据中查找的内容，就是匹配模式
  action: 在找到匹配内容时所执行的一系列命令,类似于函数，可以读参数和变量
  - 例子1，获取root用户登录,如果使用cut
    ```
    cat /etc/passwd | grep ^root | cut -d ":" -f 7
    ```
    如果使用akw
    ```
    cat /etc/passwd | awk -F : '/^root/{print $7}'
    ```
  - 例子2,输出多列
    ```
    cat /etc/passwd | awk -F : '/^root/{print $6","$7}'  //可以指定中间的打印分隔符，还可以打印多个列
    ```
  - 例子3，只显示第1列和第7列，逗号风格，并且在第一行增加列名"user,shell"，并且在最后一行添加 "at the end"
    使用 BEGIN 和 END 来操作
    ```
    cat /etc/passwd | awk -F : 'BEGIN{print "user,shell"}{print $1","$7} END{print "at the  end"}' 
    ```
  - 例子4：定义变量 ,变量引用不需要加 $ 符号
    ```
    cat /etc/passwd | awk -v i=1 -F ":"  '{print $3+i}'
    ```
##### 内置变量:
  - FILENAME  文件名
  - NR 已读的记录数(行号)
  - NF 浏览记录的域的个数(切割后,列的个数)

  例子：输出3个信息
  ```
  awk -F ":"  '{print "文件名:"FILENAME "行号:"NR "列数:"NF}' /etc/passwd
  ```
  例子：输出空行
  ```
    ifconfig | grep -n ^$
    ifconfig | awk '/^$/{print NR}'
  ```
### 综合应用

#### 发送消息
  使用`write`可以给其他终端发送消息，只要对方打开了消息接收功能
  1. 查看是否打开了消息接收功能` who -T ` ，显示的列分别为  用户名| + (表示打开) | 终端号
  2. 如果没打开，可以使用 `mesg n` 打开终端
  3. 使用命令 `write [用户名]  [终端号]` 即可以发送消息

#### 写一个发送消息的脚本
  易错点:
  1. 变量赋值=前后不能有空格
  2. if后面的 [ ] 判断条件和方括号必须 空格 分开
  
  ```
  #!/bin/bash
  #查看用户是否登录
  send_user=$(who |  grep -i -m 1 $1 | awk '{print $1}' )

  if [ -z $send_user ]
  then
          echo "用户名为空"
          echo "脚本退出"
          exit
  fi

  # 查看用户是否开启消息功能
  is_allowed=$(who -T | grep -i -m 1 $1 | awk '{print $2}' )

  if [ $is_allowed != "+" ]
  then
          echo "$1没有开启消息功能"
          echo "脚本退出"
          exit
  fi

  #确认是否有消息发送
  if [ -z $2 ]
  then
          echo "没有消息发送"
          echo "脚本退出"
          exit
  fi

  # 获取消息
  messages=$(echo $* | cut -d " " -f 2-)

  # 获取用户的终端
  user_terimal=$(who -T | grep -i -m 1 $1 | awk '{print $3}' )

  # 输出看看参数正确不
  echo "user:"$send_user
  echo "terimal:"$user_terimal
  echo "messages:"$messages


  # 发送消息
  echo $messages | write $send_user $userterimal

  if [ $? = 0 ]
  then
          echo "发送成功"
  else
          echo "发送失败"
  fi

  exit

  ```