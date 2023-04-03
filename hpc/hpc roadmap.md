[toc]

## JD

* linux
* RAID, Erasure Code, RDMA, SaaS, Software Defined Storage (SDS)
* Parallel FS, Object Storage, Network attached storage (NAS)
* SSD FS
* Distributed FS
* MPI parallel IO optimization





## 下层：计算机体系结构

* C++
* SIMD
* Profiler
  * [Linux perf 各种工具](https://www.brendangregg.com/linuxperf.html)
  * [HTTP Performance tuning](https://talawah.io/blog/extreme-http-performance-tuning-one-point-two-million/)
* 计算机体系结构
  * [现代微处理器架构 90 分钟指南](https://www.starduster.me/2020/11/05/modern-microprocessors-a-90-minute-guide/)



## 中层：编程模型和代码生成

* MPI
* CUDA
* 并行文件系统
  * Lustre
  * BeeGFS



## 上层：算法设计

xxx





## 分布式存储系统路线

一个分布式系统要学习的模块主要有：

- 元数据和配置管理中心(glusterfs 是无中心的，有一套单独的管理方式)
- 监控模块
- 客户端模块
- 生态工具
- 数据异常修复
- 高级特性
- 单机引擎
- 文件数据结构和语义
- 测试优化



以上模块的优先级和一些注意事项：

| 模块                 | 参考开源项目         | 优先级 | 注意事项                                                     |
| -------------------- | -------------------- | ------ | ------------------------------------------------------------ |
| 文件数据结构和语义   | zfs                  | 最高   | 核心模块，其他模块都是围绕该模块展开，贯穿整个职业生涯修炼过程，也是核心竞争力所在之一。 |
| 数据异常修复         | glusterfs/ceph/hdfs  | 非常高 | 重难点模块之一, 也是每个分布式系统的核心                     |
| 元数据和配置管理中心 | tikv/etcd/ceph       | 较高   | 元数据索引管理,也是重难点之一                                |
| 单机引擎             | lustre/zfs/rocksdb   | 高     | 部分分布式存储系统会根据文件系统来进行优化参数, 如lustre on zfs |
| 测试优化             | ebpf                 | 较高   | 单元测试, 集成测试,性能测试，都是必不可少的，目前分布式存储测试也是很难的，尤其是模拟硬件故障测试 |
| 客户端               | juicefs              | 较高   | 客户端的错误重试，对异常是否区分可重试与不可重试等，都是需要考虑的，也是重难点 |
| 监控                 | ceph                 | 一般   | 主要知道监控的一些重点内容,还有健康存活性判断 那些           |
| 高级特性             | zfs                  | 一般   | 部分高级特性在单机文件系统中有，但是分布式一定会存在,例如zfs动态trim和重复数据删除(dedup) |
| 生态工具             | nfs-ganesha, csi插件 | 一般   | 每家的生态工具不一定相同,同时学习资料项目较少。              |





## 补充资源

**论文**

SC（大而广），ICS，PPoPP（偏并行编程），IPDPS（系统算法都有），SPAA（并行算法与architecture）。ICPP也不错。

ISCA、MICRO、HPCA、PACT里面许多关于高性能体系结构或者编译的工作。

**课程**

比较全面的介绍（包括并行算法、并行编程）：[UC Berkeley CS267](https://link.zhihu.com/?target=https%3A//sites.google.com/lbl.gov/cs267-spr2021%3Fpli%3D1)

更偏并行算法一些的：[Georgia Tech CSE 6220](https://link.zhihu.com/?target=https%3A//omscs.gatech.edu/cse-6220-intro-hpc)

从系统优化的角度入手：[MIT 6.172](https://link.zhihu.com/?target=https%3A//ocw.mit.edu/courses/6-172-performance-engineering-of-software-systems-fall-2018/)

**docs**

[lustre wiki](https://wiki.lustre.org/Main_Page)

