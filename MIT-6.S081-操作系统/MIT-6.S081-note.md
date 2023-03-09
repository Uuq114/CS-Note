[toc]

# 相关链接

* [课程内容翻译Gitbook](https://mit-public-courses-cn-translatio.gitbook.io/mit6-s081/)
* [课程作业](https://pdos.csail.mit.edu/6.828/2021/schedule.html)
* 我的Lab仓库（todo）
* 我的Lab笔记（todo）



# Lecture 1

Q：系统调用跳到内核与标准的函数调用跳到另一个函数相比，区别是什么？

A：kernel的代码有特殊的权限，可以直接访问各种硬件，比如磁盘



**exec调用**

`exec`系统调用会从指定文件读取、加载指令，并替代当前调用进程的指令

可以传入一个命令行参数的数组：`exec("echo", argv);`

* exec调用前后，fd表示的东西不受影响
* exec不会返回，除非出错



因为exec不能返回，在shell中执行指令时，shell会先fork出child，child执行exec



**`wait`的工作原理**：

如果当前进程有任何child，并且一个child已经退出了，那么wait会返回



**IO重定向**

shell可以使用`>`、`<`进行IO重定向，原理是：
shell先fork，然后在child中改变fd指向的文件
