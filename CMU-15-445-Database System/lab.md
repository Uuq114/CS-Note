# Lab

## Project 1

背景：
conflict-free replicated data type（CRDT）
一种分布式系统中的数据结构，通过网络在多个计算机上维护多个副本。CRDT 可应用在在线文档、分布式数据库、移动应用（多设备同步数据、离线状态也能更新数据）中。特点：

- 每个节点可以独立地更新副本
- 在副本之间进行同步时，CRDT 会合并修改

CRDT 的类型：

- operation-based
  - 也称为 commutative replicated data type，commutative 意为符合交换律的。
  - 内容：副本之间在同步时只传递 update 操作，其他副本会在本地应用这些修改。
  - 要求：操作要满足交换律，所有备份操作的内容相同
- state-based
  - 也称为 convergent replicated data type，convergent 意为收敛的
  - 内容：副本在同步时发送 full local state，由一个满足交换律、结合律、幂等律的函数 merge

例如，OR-set 就是一种 CRDT，可以处理 set 中元素的增减。
在添加元素时，每个元素会打上一个独特的标识。在删除元素时，这个标识会被加到 tombstone set，而不是直接删除。OR-set 可以追踪元素的添加和删除，因此可以在删除之后重新添加元素。
在处理 “对一个元素并发的 add/remove” 时，有三种可能情况：add 赢、remove 赢、error，OR-set 会让 add 赢。
OR-set merge 的结果是确定的。
