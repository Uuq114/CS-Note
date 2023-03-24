[toc]

## 从makefile开始

### 概述

在一个工程中，不计其数的源文件按照类型、功能、模块分布在不同目录中。可以在makefile中指定编译规则，如编译的先后顺序。除了编译，makefile还可以执行其他操作，因为makefile就像shell脚本，也可以执行OS的命令。



### 程序的编译和链接

源文件=>中间代码文件（windows: `.obj`，unix: `.o`）=>可执行文件

中间文件太多时，需要对它们打包，在windows下，这样的包叫`.lib`，在unix下为`.a`



### makefile写法

makefile指定了编译和链接程序的规则

```
target: prerequisites...
	command
	...
```

target：可以是object file、可执行文件、label

prerequisites：target依赖的文件、target

command：任意的shell命令

每个makefile都需要有清空目标文件的规则：

```makefile
.PHONY: clean
clean:
	-rm edit $(objects)
```

`.PHONY`表示`clean`是一个伪目标，`-rm`表示跳过有问题的文件，`rm`后面的命令还会执行



### 在makefile中使用变量

```makefile
objects = main.o kbd.o command.o display.o \
    insert.o search.o files.o utils.o

edit : $(objects)
    cc -o edit $(objects)
```

makefile中的变量和c++中的宏一样，是直接用字符串替换的。如果变量的值包含通配符，比如`objects=*.o`，如果需要让通配符展开，要用`wildcard`



### make自动推导

make看到一个`xxx.o`时，会自动把`xxx.c`加到依赖关系中



### 指定目录路径

`VPATH`变量指定了路径，makefile会去对应的目录找文件，目录之间用`:`分隔

```makefile
VPATH = src:../headers
```

还可以使用`make vpath`指定路径：

```bash
vpath %.o ../headers
```



### 静态模式

更容易地定义多目标的规则：

```makefile
<targets ...> : <target-pattern> : <prereq-patterns ...>
    <commands>
    ...
```

target-pattern: 指明了targets的模式

prereq-patterns: 指明了依赖的模式 

例如：

```makefile
objects = foo.o bar.o

all: $(objects)

$(objects): %.o: %.c
    $(CC) -c $(CFLAGS) $< -o $@
```

`$<`表示第一个依赖文件，`$@`表示target。

上面的展开就等价于：

```makefile
foo.o: foo.c
	$(CC) -c $(CFLAGS) foo.c -o foo.o
bar.o : bar.c
	$(CC) -c $(CFLAGS) bar.c -o bar.o
```



### 自动生成依赖关系

编译器自动寻找源文件中包含的头文件

```bash
cc -M main.c
```

一般用下面的，不会列出来标准库的文件：

```bash
cc -MM main.c
```

GNU组织建议把编译器为每一个源文件的自动生成的依赖关系放到一个文件中，为每一个 `name.c` 的文件都生成一个 `name.d` 的Makefile文件， `.d` 文件中就存放对应 `.c` 文件的依赖关系。

可以在makefile中添加根据`xxx.c`源代码文件生成`xxx.d`依赖文件的规则，再把`xxx.d`include到makefile中





