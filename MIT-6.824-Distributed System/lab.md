# MIT 6.824: Distributed System

## 课程资料

原来的课程编号换成了 6.5840，因此官网也更新了：<http://nil.csail.mit.edu/6.5840/2024/>。我使用的是 Spring 2024 版本。

Syllabus：<http://nil.csail.mit.edu/6.5840/2024/schedule.html>

## Lecture 1

### 预习 Mapreduce

总结：

- MapReduce 是一种处理和生成大数据集的编程模型，MapReduce 程序是天然并行的，系统会处理并行、容错、数据管理、负载均衡，用户只需要关心关键的 map/reduce 逻辑
- `map` 函数会对每条 record 生成中间 kv 对，并且生成中间 kv 对
- `reduce` 函数会合并 key 相同的中间 kv 对

举例，现在需要大量文件中每个单词出现的次数，那么编写的 map/reduce 函数是：

- map：对每个文件，生成中间 kv 对 `(word, 1)`。
- reduce：聚合相同 key 的中间 kv 对，生成 kv 对 `(word, result)`

![alt text](img/image.png)

我感觉这里 reduce 应该表示：`(k2, list(k2, v2)) -> (k2, v3)`

一些应用场景：

- 文本匹配
- url access 统计
- reverse web-link graph，web 页面跳转统计
- inverted index，搜索 word 在哪些文件中出现过
- distributed sort，后面详细讨论

MapReduce 的正确实现依赖于具体的环境，本文介绍的实现基于 Google 的环境————以太网互联的大规模消费级 PC 集群：

- 每台机器为双核 x86 处理器，Linux 系统，2-4GB 内存
- 网卡为 100Mbps 或者 1Gbps，但是实际带宽比这个低
- 一个集群有成百上千台机器，因此节点故障很频繁
- 存储使用机器上的磁盘，分布式文件系统用于管理这些磁盘的数据、保证可靠性
- 用户通过调度系统提交作业，每个 job 包含一些 task，task 会被提交到集群的可用机器上

MapReduce 执行流程：

![alt text](img/image-1.png)

- 输入被划分成 M 个 16-64MB 大小的块，中间输出被划分成 R 个块
- master 负责将工作分配给 worker，一共有 `M+R` 个 task
- map worker 会读取输入，将中间 kv 对写到内存中
- 内存里的中间 kv 对会周期性地被写到 local disk，并且文件位置会被传回 master
- reduce worker 收到中间文件的位置信息之后，会远程读这些数据。之后会按照 intermediate key 排序。排序可以将相同 key 的值聚集在一起，从而 reduce 函数可以按顺序处理 kv 对。如果数据太多，还需要使用 externally sort。接着，reduce worker 会遍历已排序的数据，对 kv 对执行 reduce 函数，将结果 append 到输出文件

最后会产生 R 个文件，一般不会组合成一个文件。用户可能直接将 R 个文件作为下一个 MapReduce 应用的输入，或者是其他分布式应用的输入

每个 task 有三种状态（idle/in-progress/completed）

容错：

- worker：
  - master 通过心跳机制探活 worker，worker 一段时间不响应会被判断为 failed。
  - failed worker 上的 completed map task 和 in-progress map/reduce task 会被标记为 idle 并被其他 worker 重新执行。completed reduce task 不会重新执行，因为输出已经写到 GFS 了，而 map task 的输出在 local disk 中。
  - 对于节点故障导致的 map task 重复执行的情况，所有 reduce worker 会被通知 reexecution，这样才能在新 worker 上读数据
- master：
  - master 需要维护的状态有：每个 map/reduce task 的状态、每个 wroker 的状态、completed map task 的输出文件的位置和大小
  - 一个 master 故障的概率不高，如果要做容错可以定期保存 checkpoint 状态
- semantics：
  - 保证分布式 MapReduce 框架的计算结果和 non-faulting 串行程序的结果一致
  - atomic commits of map/reduce。这块没太看懂

局部性：

GFS 管理的输入数据存储在 local disk，不同机器上是有副本的，可以用这一点节省网络带宽。MapReduce 在调度 map task 时，会尝试在包含输入数据副本的 worker 上调度任务，如果无法实现，会尝试在靠近副本位置的 worker 上调度任务（例如同一交换机下）。效果是，大多数数据都是本地读取的

长尾优化：

- 现象：最慢的 map/reduce task 会影响整体完成时间，
- 解决方法：MapReduce 操作接近完成时，Master 会调度剩余 in-progress 任务的备份执行，无论主任务还是备份任务完成，该任务都会被标记为已完成。

Combiner 函数：

某些情况下，中间产生的 kv 对有显著重复，例如 word 计数中会出现很多 `(the, 1)`，这时可以用 combiner function 在网络传输数据之前做 partial merging。

测试部分：

省略

