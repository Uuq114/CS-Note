[toc]



## 简介

课程网站：http://csapp.cs.cmu.edu/

Lab assignments：http://csapp.cs.cmu.edu/3e/labs.html



## 1 Course Overview

* 计算机中数字的表示方法
* 机器语言、程序的解释和运行
* 内存模型
* 网络编程
* 并行编程、同步



## 2 Bits, Bytes and Integers

位运算可以用于集合的表示和运算：

```
和bitmap类似,1101表示集合{0,2,3},1000表示集合{3}
那么两个集合的交集可以用&表示
```

有符号的整数最高位是负数：

```
有一个5位的有符号数，每一位代表的数分别是：-16,8,4,2,1
那么，-10可以表示位10110
所以，5位的有符号数能表示的最小值是10000，最大值是01111
```

算术扩展，需要将符号位复制到所有扩展的位上：

```
一个5位的有符号数，原来是10110
现在扩展到6位：110110，按照前面的权重计算方法，-16=-32+16
扩展到8位：11110110，-16=-2^7+2^6+2^5+2^4
算术右移也是类似的
```



字节序：

* 大端（big endian）：高位字节在低地址，大端模式和字符串的存储模式类似
* 小端（little endian）：低位字节在低地址



## 3 Floating Point

浮点数的也可以用二进制表示：

![float.drawio](lecture/float.drawio.png)

上面的表示法有局限性

* 像`0.3`这样的数就不能用有限的位数表示，得表示为部分位循环的`0.0101[01]...`这种
* 当小数点的位置固定的时候，能表示的数的范围就固定了，而且表示范围和精度不能兼得

鉴于上面的局限性，**浮点**就是可移动的小数点



**浮点表示法**

类似于科学计数法，形式为：$(-1)^s \ M \ 2^E$

s 是 sign bit，决定正负；M 是尾数，一个二进制小数，范围是$[1.0,2.0)$；E 是阶码

 在编码中的形式为：

```
====================
|| s | exp | frac ||
====================
```

exp field 编码了 exp，但是不等于 E；frac field编码了 M，但是不等于 M

在 float32 中，s-exp-frac 的位数为 1-8-23 bit；在 float64 中，s-exp-frac 的位数为 1-11-52 bit



**单精度浮点数**

![image-20230613000056095](lecture/image-20230613000056095.png)

In normalized value:

* exp field is not 11...1 or 00...0
* $E=Exp-bias$
  * Exp: exp field unsigned value, bias: $2^{k-1}-1$, k: exp bits
  * single precision bias = 127, E in -126...127
  * double precision bias=1023, E in -1022...1023
* frac has implied leading `1`, $M=1.xxxx_2$
  * M in $1.0$...$2.0-\epsilon$

把浮点数设计成这种`符号位-阶码-尾数`的编码，是为了<u>方便浮点数之间的比较</u>。本来 exp 可能是负的，但是加了 bias 之后，encoded exp field 只可能是无符号的整数，方便比较。



In denormalized value:

* exp field = 00...0
* E = 1 - bias
* frac has implied leading `0`, $M=0.xxxxx_2$

当 exp field 和 frac field 全为 0 时，因为符号位 s，可能有`+0`和`-0`



Special values:

* exp field = 11...1
* $\infty$: exp = 11...1, frac = 00...0
* NaN: exp = 11...1, frac is not 00...0



把 norm value 和 denorm value 表示在数轴上的话，越靠近 0 的地方数字是越密集的，可以这样理解，在 exp field 固定时，frac field 增长的速度要乘以一个权重才等于整个数字增长的速度，这个权重就是由 exp field 决定的，所以数字越大在数轴上越分散。



**Rounding**

四舍六入五成双

IEEE 浮点数使用的舍入方式是：nearest even，向最近的偶数舍入



## 4 Machine-Level Programming I: Basics

object code in binary form 和 assembly code 是一一对应的关系



**Concepts**

Intel x86 processors:

* Complex Instruction Set Computer (CISC), Reduced ... (RISC)

ARM: Acronym RISC Machine

Architecture (Instruction Set Architecture, ISA): 定义了处理器指令集和规范。例如 Intel 的 x86、IA32、x86-64和 ARM 的 ARMv7、ARMv8

Microarchitecture: ISA implementation 



**C code -> object code**

![compile.drawio](lecture/compile.drawio.png)



**Assembly Characteristics**

Data type:

* Integer data
* floating point data
* code: byte sequence

Operations:

* arithmetic function on register/memory
* transfer data between register & memory
* transfer control



disassemble binary file: `objdump -d sum > sum.d`

gdb也可以反汇编



**数据格式**

| Intel 数据类型 | 汇编代码后缀 | 长度（字节） |
| -------------- | ------------ | ------------ |
| byte           | b            | 1            |
| word           | w            | 2            |
| double word    | l            | 4            |
| quad word      | q            | 8            |
| 浮点数单精度   | s            | 4            |
| 浮点数双精度   | l            | 8            |

双字 double word 也叫 long word，所以后缀是 l

浮点数用的是一组不同的指令和寄存器



**Register**

`%rxx` 64bit, `%exx` 是 `%rxx` 的低 32bit

程序计数器（PC）在`%rip`

常用的寄存器：

| 寄存器             | 功能        |
| ------------------ | ----------- |
| rax                | 返回值      |
| rsp                | 栈指针      |
| rdi, rsi, rdx, rcx | 第1-4个参数 |

部分指令只会操作寄存器的部分位，生成1、2字节的指令不会影响寄存器的其他位，生成4字节的指令会将高4个字节置0



移动数据：`movq <source>, <dest>`

带偏移量的：`movq 8(%rbp), %rdx`



取有效地址，load effective address: `leaq <src>, <dst>`

src是一个address mode expression，lea`会把src的值设置为expression的值，src只能是register

```assembly
leaq (%rdi, %rdi, 2), %rax		# 3 * x
salq $2, %rax	# 12 * x
```



## 5 Machine-Level Programming II: Control

register:

* temp data (rax...)
* runtime stack (rsp...), rsp: current stack top
* current code control point (rip...). rip: instruction pointer
* recent test status (CF, ZF, SF, OF...)



condition code:

CF: carry flag (unsigned), ZF: zero flag, SF: sign flag (signed), OF: overflow flag(signed)

`cmp`和`test`，这两个指令可以对上面几个寄存器赋值

`sub <src>, <dst>`是执行减法的命令，把结果存到`dst`。像这种`<opr> <src>, <dst>`的模式，因为最后结果是放到`dst`的，所以后面的数都是被减数。



do-while loop, while loop, for loop 都是通过`jmp`、`test`实现的



**indirect jump**

switch 因为有很多 case，所以有一个 jump table 的结构，jump table 里面的元素是 `case <val>` 下面的 block 开始的地方，比如`.L8`这种。所以实际的地址跳转是：start -> jump table arg -> block

`jmp`首先会和最大的`case <val>`比较，而且用的是无符号数的`ja`（jump above），这样可以同时让小于 0 的和 大于最大值的都跳到 default block

> 如果`case <val>`的范围很大或者为负值怎么办？
>
> 为负值：会加一个 bias，避免索引出负值
>
> 范围很大：`case <val>`比较稀疏。中间的值需要建表吗？编译器会优化成 if-else 结构，甚至会用二分搜索让复杂度降到 logn



## 6 Machine-Level Programming III: Procedures

函数之间互相调用的时候，需要：

* passing control，开始调用和返回的时候，转移控制权
* passing data，传递参数和返回值，中间变量要释放



栈：函数的调用和返回，对应入栈和出栈

栈的底部是高地址，向低地址增长。所以，创建栈顶元素时，先减少栈指针，再写入



调用`callq`时：

* 将`callq`下一行的指令地写入栈顶
* `%rip`被替换为`callq`的参数，转移控制流

调用`ret`时：

* 用`%rsp`获取栈顶值，替换`%rip`

* 增加`%rsp`



函数的返回值保存在`%rax`中



**栈帧**

存在多级函数调用时，栈里面会有多个元素，每个元素称为栈帧，栈帧存储了函数的状态。栈帧的范围通过`%rbp`和`%rsp`标识

因为寄存器里面的值可能被其他函数改变，所以function caller有时会将可能被修改的register（如`%rbx`）推到栈里面



## 7 Machine-Level Programming IV: Data
