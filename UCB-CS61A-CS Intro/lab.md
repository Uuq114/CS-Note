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