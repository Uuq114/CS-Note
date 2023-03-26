[toc]

* UCB CS61A: Structure and Interpretation of Computer Programs
* 课程网站：https://inst.eecs.berkeley.edu/~cs61a/sp21/
* 中文翻译：https://composingprograms.netlify.app/



### 1 使用函数构建抽象

**帧**

求解表达式的环境由帧（frame）的序列组成，每个帧包含一些绑定，将名称与对应的值（字面量或者指向对象）关联。

![image-20230325193112312](assets/image-20230325193112312.png)



![image-20230325193055781](assets/image-20230325193055781.png)

全局帧（global frame）只有一个。赋值和导入语句会将条目添加到当前环境的第一帧。



函数有两个名称：

* 内在名称：作为函数定义的一部分
* 绑定名称：在帧中出现的名称

不同的名称可能指同一个函数，但是一个函数只有一个内在名称

在求值过程中起作用的是绑定名称

```python
f = max
max = 3
res = f(2,3,4)
max(1,2)	# TypeError: 'int' object is not callable
```



调用用户定义的函数会引入局部帧（local frame），它只能访问该函数

在局部帧中，形参和值绑定

```python
from operator import mul
def square(x):
    return mul(x, x)
square(-2)
```

![image-20230325195548695](assets/image-20230325195548695.png)



**lambda表达式**

一个lambda表达式的计算结果是一个函数，它仅有一个返回表达式作为主体，不允许使用赋值和控制语句

```python
def compose1(f, g):
    return lambda x: f(g(x))
```

lambda 表达式的结果称为 lambda 函数（匿名函数）。



**函数装饰器**

使用高阶函数作为执行`def`函数的一部分，称为装饰器

```python
>>> def trace(fn):
        def wrapped(x):
            print('-> ', fn, '(', x, ')')
            return fn(x)
        return wrapped
>>> @trace
    def triple(x):
        return 3 * x
>>> triple(12)
->  <function triple at 0x102a39848> ( 12 )
36
```

`trace`就是一个高阶函数

通过注解`@trace`，`triple`这个名字绑定到了`trace(triple)`这个函数上面

```python
>>> def triple(x):
        return 3 * x
>>> triple = trace(triple)
```



## 2 使用数据构建抽象