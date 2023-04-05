[toc]

## hw2

丘奇数（church numeral）：一种只使用函数来表示自然数的方法

https://zhuanlan.zhihu.com/p/267917164





## hw3

硬币兑换问题，要求使用递归的方法

```python
# ways of dividing n into several parts, the greatest part is m
def count_partition(n, m):
    ...
    return count_partition(n - m, m) + count_partition(n, m - 1)
```

两个例子：`cp(4,2)`和`cp(6,3)`

https://juejin.cn/post/7205161917420929061





匿名递归阶乘：

https://blog.csdn.net/qq_42103298/article/details/123773235

https://zhuanlan.zhihu.com/p/506640384



## proj cat

创建嵌套列表：

```python
a = [[]] * 3
a[0].append(1)	# [[1], [1], [1]]
```

上面这种情况是因为`a[0]`、`a[1]`、`a[2]`的地址是一样的，修改一个子元素会导致其他的也会被修改

这总情况可以用列表推导解决：

```python
a = [[] for _ in range(3)]
a[0].append(1)	# [[0], [1], [1]]
```



## lab5

输入一棵树`t`以及一个列表`leaves`，要求对每个叶子节点添加一个branch，其值为`leaves`中的元素

sprout_leaves

```python
def sprout_leaves(t, leaves):
    """Sprout new leaves containing the data in leaves at each leaf in
    the original tree t and return the resulting tree.

    >>> t1 = tree(1, [tree(2), tree(3)])
    >>> print_tree(t1)
    1
      2
      3
    >>> new1 = sprout_leaves(t1, [4, 5])
    >>> print_tree(new1)
    1
      2
        4
        5
      3
        4
        5

    >>> t2 = tree(1, [tree(2, [tree(3)])])
    >>> print_tree(t2)
    1
      2
        3
    >>> new2 = sprout_leaves(t2, [6, 1, 2])
    >>> print_tree(new2)
    1
      2
        3
          6
          1
          2
    """
    "*** YOUR CODE HERE ***"
	    if is_leaf(t):
        return tree(label(t), [tree(i) for i in leaves])
    else:
        return tree(label(t), [sprout_leaves(branch, leaves) for branch in branches(t)])

```



字符串的拼接之join：





## lab6

yield

```python
def scale(it, multiplier):
    """Yield elements of the iterable it multiplied by a number multiplier.

    >>> m = scale([1, 5, 2], 5)
    >>> type(m)
    <class 'generator'>
    >>> list(m)
    [5, 25, 10]

    >>> m = scale(naturals(), 2)
    >>> [next(m) for _ in range(5)]
    [2, 4, 6, 8, 10]
    """
    "*** YOUR CODE HERE ***"
    for i in it:
        yield i * multiplier
```



## lab10

`scheme`中声明`lambda`匿名函数的时候`x`也要括号括起来：

```scheme
(define (make-adder num) (lambda (x) (+ num x)))
```

将`list`中等于`item`的元素去掉，返回新的`list`：

```scheme
(define (remove item lst)
    (cond ( (null? lst) '() )
          ( (= (car lst) item) (remove item (cdr lst)) )
          ( else (cons (car lst) (remove item (cdr lst))) )
    )
)
```

这里注意`cons`和`list`的区别：

```scheme
(define a (list 2 3))
(cons 1 a)	; (1 2 3)
(list 1 a)	; (1 (2 3))
```

