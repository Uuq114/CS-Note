# Lecture

<!-- TOC -->

- [Lecture](#lecture)
  - [Lecture 1 - Course Overview \& Relational Model](#lecture-1---course-overview--relational-model)

<!-- /TOC -->

## Lecture 1 - Course Overview & Relational Model

Overview
这门课程是关于 database management system 的 design/implementation 的一门课程

database 对真实世界中的数据做了建模，将内在相关的数据有序组织起来

数据库和普通的存储数据的文件（flat file）有什么区别？

Q：假设我们需要存储一个音乐库信息，包括artist和album类别，并且直接用csv存储（`artist=name+year+country`，`album=name+artist+year`），那么可能有以下问题：

数据完整性：

- 一张专辑对应多个艺术家
- 有人尝试将某个album year改为invalid string
- 需要删除一个artist，而他名下有一些album

实现：

- 如何找到某条记录
- 同时有多个线程需要读写同一个csv文件

持久性：

- 当程序正在更新一条记录时，机器故障了？
- 为了保证高可用，需要复制DB到多台机器？

