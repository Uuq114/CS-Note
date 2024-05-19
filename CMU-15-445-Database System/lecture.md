# Lecture

<!-- TOC -->

- [Lecture](#lecture)
  - [Lecture 1 - Course Overview \& Relational Model](#lecture-1---course-overview--relational-model)
  - [Lecture 2 - Modern SQL](#lecture-2---modern-sql)
  - [Lecture 3 - Database Storage (Part 1)](#lecture-3---database-storage-part-1)

<!-- /TOC -->
<!-- /TOC -->
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

## Lecture 2 - Modern SQL

SQL操作的对象是bag/multiset而非set，即db中是允许有重复行的

aggregate：
聚合操作，从a bag of tuples得到single value的操作，例如`AVG`、`MIN`、`MAX`、`SUM`、`COUNT`.

`DISTINCT`：聚合操作基本只能用在`SELECT`中。`AVG`、`SUM`、`COUNT`支持`DISTINCT`去重。
`GROUP BY`：将tuple投影到subset，即分组。注意在`SELECT`输出结果中出现的非聚合列**必须**出现在`GROUP BY`中。

```sql
SELECT AVG(s.gpa), e.cid, s.name
  FROM enrolled AS e JOIN student AS s
    ON e.sid = s.sid
  GROUP BY e.cid, s.name
```

这里`e.cid`和`s.name`必须在`group by`中出现，因为它们不是聚合函数中的字段，查出来的值可能不唯一。加到`group by`里之后，相当于把整个`e.cid, s.name`作为group的标准

![alt text](img/image-4.png)

`HAVING`
在聚合的基础上的filter，类似于对`GROUP BY`的`WHERE`

![alt text](img/image-5.png)

操作字符串

- `LIKE`匹配字符串，`%`匹配任意长度的字符串，`_`匹配任意字符
- 有的DBMS有内置的函数，比如`SUBSTRING`、`UPPER`
- 拼接字符串，使用`||`，或者内置的`CONCAT`

处理输出
`ORDER BY`排序，`LIMIT`指定tuple数量和offset

嵌套查询
`ALL`、`ANY`、`IN`（和`ANY`等价）、`EXISTS`

Window Function

![alt text](img/image-6.png)

```sql
select *, row_number() over () as row_num
from enrolled
```

![alt text](img/image-7.png)

`OVER`指定了将tuple分组的方式。使用`PARTITION BY`可以指定group

```sql
select *, row_number() over (partition by cid) as row_num
from enrolled
```

![alt text](img/image-8.png)

如果此时有`OEDER BY`，是在每个group内部排序

举例，从选课表获取成绩第二高的学生，包括所有课程

```sql
select * from (
  select *, rank() over (partition by cid order by grade asc) as ranking
    from enrolled
) as ranks
where ranks.ranking=2
```

临时表

![alt text](img/image-9.png)

可以定义一个临时的表，在后面的SQL再使用

## Lecture 3 - Database Storage (Part 1)

![alt text](img/image-10.png)

potpourri，大杂烩

Disk-based architecture
DBMS假设DB的主要存储是disk。DBMS的组件需要在volatile和non-volatile storage上管理数据

一些常用的数字：

- L1 cache，1 ns
- L2 cache，4 ns
- DRAM，100 ns
- SSD，16000 ns，0.016 ms
- HDD，2 ms
- Network storage，50 ms
- Tape，1000 ms，1 s

sequential/random access

non-volatile storage上的random access比sequential access慢很多

因此DBMS希望能尽量使用sequential access：

- 减少向random page写的次数
- 一次分配多个page，称为一个extent

DBMS设计目标：

- 让DBMS可以管理大小超过available memory的DB
- 因为磁盘读写开销很大，需要控制向磁盘的读写次数
- 因为随机访问的开销大于顺序访问，因此需要尽量用顺序访问

面向磁盘的DBMS设计：

![alt text](img/image-11.png)

一个问题：
Q：
在上面的图中，DBMS需要自己在memory和disk之间管理内存。OS提供的mmap（memory mapping）可以将磁盘上文件的内容映射到进程的address space中，而进程则可以跳转到任何offset。由OS决定何时将page移入/移出memory。因此如果DBMS使用mmap，就可以让OS来管理所有的数据，DBMS自己并不需要“写入”任何数据。为什么不让OS来替DBMS管理数据呢？

A：
如果DBMS是只读的，那确实可以用mmap。但是如果有写入，尤其是有多个thread需要访问mmap-ed file的时候，情况会很复杂。OS只会替换脏页，而DBMS在执行事务时需要保证多个写执行的顺序。

使用mmap io带来的问题：

- Transaction safety。OS可能在任何时候刷新脏页
- IO stall。DBMS不知道哪些page在内存中。在page fault时thread需要等待。
- Error handling。访问mmap-ed file可能产生`SIGBUS`，DBMS需要处理
- Performance issue。OS data structure contention

有一些syscall可以告诉OS如何管理page，例如`madvise`、`mlock`、`msync`，但是用这些来保证OS正常工作，还不如自己管理内存。

让DBMS自己管理数据的优点：

- flush dirt pages to disk in correct order
- prefetch
- buffer replacement policy
- thread/process scheduling

DBMS file storage的两个问题：

- 如何在disk file上表示DB（本次lecture）
- 如何管理memory，以及在disk-memory之间移动数据

Storage Manager
负责维护database file。files包括很多的pages，storage manager负责数据的读写、追踪剩余的空间

Database page
page是一个固定大小的block，可以存储tuple、metadata、index、log等。有的DBMS要求page是self-contained的。每个page有唯一标识。DBMS将page ID和物理存储对应起来

DBMS中的各种page：

- hardware page，一般4KB
- OS page，一般4KB
- DB page，512B-16KB

hardware page是存储设备（disk等）能保证failsage write（原子写？）的最大的block

Page storage architecture
不同的管理page的方式：

- heap file
- tree file
- sequential/sorted file
- hashing file

这些组织方式只到page一层，和page的内部结构无关

Heap file
无序page的集合。tuple以随机顺序存储。
支持操作：create/get/write/delete page，page iterator用于顺序遍历
如果DB只有单个文件，就比较容易定位page，`offset = page# * pagesize`
如果DB有多个文件，需要额外记录：file-page对应关系，哪个file有剩余空间

Heap file: directory page
在heap file organization中，除了普通的data page，还有directory page，用来记录data page在DB file的位置，以及剩余空间（每个page的free slot、free page list）。（page metadata）

因为引入了额外的metadata，DBMS需要保证directory page和data page是同步的。

Page header
page header中是page metadata，记录和page content有关的信息。如page size、checksum、DBMS version、transaction visibility info、compression等。有的DBMS要求page是self-contained的。

![alt text](img/image-12.png)

Page layout
组织page data的方式。

- tuple-oriented。
- log-structured。

对于只存储tuple的page。一种最简单的page layout是：开头存储page数目，后面一直追加新的tuple

![alt text](img/image-13.png)

这种方式的问题：

- 如何删除tuple？
- 如果tuple长度是可变的？

上面的page layout的改进版本是slotted pages。
slot array将slot映射到tuple的offset。（从这个图看，tuple是倒着分配空间的？）

![alt text](img/image-14.png)

每个tuple都有一个unique ID。最常见的计算方式是：`page_id + offset`，例如sqlite和oracle中的rowid，postgresql的ctid。

Tuple layout
tuple的内容主要是attribute的type和value。tuple包括tuple header和tuple data。
tuple header主要有：

- visibility info，用于concurrency control
- bitmap for null values

tuple header并没有schema的metadata信息。（attribute name这种存在外面的table metadata里面？）

tuple data是attribute value。它们的顺序是创建表的时候，attribute的顺序。
