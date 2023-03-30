# Unix-like file systems

[toc]

## 写在前面

这篇笔记主要内容是 Linux 上的文件系统，讨论文件系统的基本设计

Ken Thompson 先实现了文件系统，在此基础上实现的 Unix



## 1 抽象

**对文件的抽象：**

* a sequence of bytes
* 文件的格式取决于用户如何解释这个文件

10个基本的系统调用：

<img src="assets/image-20230329231559758.png" alt="image-20230329231559758" style="zoom: 50%;" />



**文件系统的抽象：**

* 从文件名到文件的映射

  * 文件的元数据（权限、类型、user、group、创建时间、修改时间、...）

  * 文件的内容



**Unix文件的抽象：使用 inode 作为桥梁**

filename => inode => File

inode 是一个整数，可以看成是 FS 里面的 primary key

存在多个文件名指向一个 inode 的情况（硬链接，hard link）



**磁盘的抽象：块设备**

磁盘读写的单位是扇区（sector），扇区大小为 4 KB。

块设备具有线性连续的块空间，每个块有一个下标。块的大小有 1 KB（老设备）、2 KB、4 KB（现在更常见）的。



**文件系统在块设备和文件之间的角色**

基于块设备，文件系统提供了操作文件的接口

![image-20230329235447216](assets/image-20230329235447216.png)



## 2 磁盘



















































