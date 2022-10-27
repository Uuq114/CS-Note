# mtail 

[toc]

## 简介

mtail是用来从应用日志提取数据，并上报给时间序列数据库，用于数据可视化和告警的

应用 --> mtail --> 监控系统



## 执行

对接收的每一行日志，运行所有的程序，每个程序运行一次：
```pseudocode
for line in lines:
  for regex in regexes:
    if match:
      do something
```



## mtail程序结构

exported variable、pattern-action声明、optional decorator定义

```
exported variable

pattern {
  action statements
}

def decorator {
  pattern and action statements
}
```



### exported variable

上报的变量在这里定义：

* counter和gauge是type

  counter：适用于单调递增的变量，用于计数类，例如传输的字节数

  gauge：任何时候都可以随便赋值，例如某时刻的queue length

  histogram：记录event在某个dimension下的频率，例如latency，只能识别float

  ```
  counter lines_total
  gauge queue_length
  ```

* as可以定义上报的变量名：as

  ```pseudocode
  counter lines_total as "line-count"
  ```

* 把变量变成多维数组：by

  ```pseudocode
  counter bytes by operation, direction
  counter latency_ms by bucket
  ```

* 定义不需要上报的变量，用于存储中间信息：hidden

  这是在处理不同行的时候，共享数据的唯一方法

  ```pseudocode
  hidden counter login_failures
  ```



### pattern-action声明

条件，以及对应的行为：

```
COND {
	ACTION
}
```

`COND`可以是正则表达式、条件表达式，以及两者的结合：

```pseudocode
/foo/ {
	ACTION1
}
/foo/ && var > 0 {
	ACTION2
}
```

正则表示式可以被声明为`const`，从而重复使用：
```
const PREFIX /^\w+\W+\d+ /

PREFIX {
	ACTION1
}

PREFIX + /foo/ { # 匹配PREFIX和foo
	ACTION2
}
```

mtail支持的运算符：

```
关系运算符
< less than
<= less than or equal
> greater than
>= greater than or equal
== is equal
!= is not equal
=~ pattern match
!~ negated pattern match
|| logical or
&& logical and
! unary logical negation

算术运算符
| bitwise or
& bitwise and
^ bitwise xor
+ addition
- subtraction
* multiplication
/ division
<< bitwise shift left
>> bitwise shift right
** exponent

用在exported variable上的运算符
= assignment
++ increment
+= increment by
-- decrement
```



## mtail内置函数

不会影响state的函数：

```
int(x)
float(x)
string(x)
strtol(x, y)
```

会影响state的函数：

```
getfilename() 获取文件名
settime() 设置current timestamp register
```











## 参考

* https://github.com/google/mtail/blob/main/docs/Language.md