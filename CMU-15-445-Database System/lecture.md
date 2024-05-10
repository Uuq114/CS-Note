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

DBMS
可以在遵守data model前提下定义、增删、查询、管理数据

Data model
DB中数据的概念的集合

Schema
给定data model之后，对特定数据集合的描述

常见的data model：

- Relational (most DBMS)
- NoSQL
  - key/value
  - graph
  - document/object
  - wide-column/column-family
- Machine Learning
  - array/matrix/vector
- Obselete/Legacy/Rare
  - hierarchical
  - network
  - multi-value

Relational model
为了减少维护的开销，relational model定义了一种database abstraction

relational model设计标准：

- 用简单的数据结构存储数据
- 物理存储细节由DBMS实现决定
- 通过高级语言来访问数据，由DBMS来决定最佳执行策略

relational model组成部分：

- structure：db的relation定义，内容
- integrity：保证db的数据满足constraint
- manipulation：访问、修改db内容的接口

relation
一个无序集合。这个集合包括多个entity attribute的关系。例如`Artist(name,year,country)`

tuple
一个集合。集合包括relation中的attribute values。

primary key
relation的primary key可以作为tuple的唯一标识

foreign key
foreign key表示一个relation的attribute是另一个relation的primary key
例如有三个表：

```pesudo
ArtistAlbum(artist_id, album_id)
Artist(id, name, year, country)
Album(id, name, year)
```

那么，`ArtistAlbum`中的`artist_id`就是foreign key，它引用了`Artist`表中的primary key作为自己的字段

Data Manipulation Language(DML)
从DB存取数据。

- procedural，过程式。描述了数据操作的步骤和流程。例如早期的数据库DML。
- non-procedural，非过程式（声明式）。只描述希望达成的结果，而不关注步骤。例如SQL。

Relational algebra
其中，procedural也称relational algebra 关系代数。关系代数中有一些运算符，用运算符对tuple进行计算，可以操作数据。一些运算符：select、projection、union等

![alt text](img/image.png)

![alt text](img/image-1.png)

![alt text](img/image-2.png)

关系代数的表达式其实还是描述了查询数据的具体操作，和前面提到的data model设计标准还是有差距。
relation model的设计和DML的设计是分开的，并不依赖具体的DML实现。
SQL是relational model DML的事实标准

一些拓展
document/object data model发展很快。
在document data model中，数据的层级由object直接体现：

![alt text](img/image-3.png)

elastic中好像就是这种结构。