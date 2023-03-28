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

