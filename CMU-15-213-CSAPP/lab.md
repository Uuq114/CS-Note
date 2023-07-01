[toc]

CSAPP lab webpage: http://csapp.cs.cmu.edu/3e/labs.html

- [x] data lab: http://csapp.cs.cmu.edu/3e/datalab.pdf
- [ ] bomb lab: http://csapp.cs.cmu.edu/3e/bomblab.pdf
- [ ] todo



## data lab

xxx



## bomb lab

**一些 GDB 使用方法**

运行

- `gdb bomb` 使用 gdb 调试可执行文件 `bomb` √
- `r` run，运行程序，遇到断点处停止，等待用户输入下一步指令 √
- `c` continue，继续执行，到下一个断点(或程序结束) √
- `q` quit, 退出 gdb 调试 √
- `si` 单指令执行，即每次只执行一条指令，结合"交互模式下直接回车的作用是重复上一指令，对于单步调试非常方便" √
- `n` next，单步跟踪程序，遇到函数调用时，不会进入函数体内部
- `s` step，单步调试，遇到函数调用时，会进入函数体内部
- `until` 当你厌倦了在一个循环体内部单步跟踪时，这个命令可以运行程序直到退出循环体
- `until + 行号` 运行至行号处

设置断点

- `b n` break n，在第 n 行设置断点 √
- `b func` 在函数 func() 的入口处设置断点 √
- `b *地址值` 在地址值处设置断点，例如，`b *0x401460` √
- `i b` info b, 显示当前程序的断点设置情况，会给出各个断点的序号，类型等信息 √
- `delete 断点号n` 删除第 n 个断点(从 `info b` 中得出断点序号) √
- `disable 断点号n` 暂停第 n 个断点
- `enable 断点号n` 开启第 n 个断点
- `delete breakpoints` 删除所有断点

分割窗口

- `layout regs` 显示寄存器和反汇编窗口 √
- `layout asm` 显示反汇编窗口 √



**phase 2**

