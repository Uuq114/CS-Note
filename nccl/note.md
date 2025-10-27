# Nvidia Collective Communication Library (NCCL)

NCCL 设计目标：使用 NVLink/PCIe/IB 等技术，让 GPU-GPU 通信达到高带宽、低延迟

下面以四个部分介绍 NCCL 的实现：

- 总览，介绍 API 结构、communication channel 管理
- 介绍通信协议（Simple/LL/LL128）
- NCCL data-transfer 模型分析
- NCCL 集合通信算法分析

## NCCL Overview

### NCCL API

主要给用户提供四种函数：

- communicator 管理
  - 所有参与通信的 GPU 都会维护一个 communicator 对象，用来调用 NCCL。用户需要先初始化一个 communicator，然后定义通信的 GPU 集合
  - 初始化。如果是单进程 / 线程，调用 `ncclCommInitAll`。如果是多进程 / 多线程，每个进程需要调用 `ncclCommInitRank` 并指定唯一标识号
  - 结束释放资源。`ncclCommDestroy` 安全结束，待处理通信结束后再清理。`ncclCommAbort` 立即结束，用于错误恢复、避免死锁
- 集合通信
  - 集合操作。`ncclAllReduce`、`ncclBroadcast`、`ncclReduce`、`ncclAllGather`、`ncclReduceScatter`，五种集合操作
- 点对点通信
  - `ncclSend`、`ncclRecv`
- 组调用
  - `ncclGroupStart`、`ncclGroupEnd`。中间包裹一系列 NCCL 调用。可以降低启动开销

### 启动策略

在多 GPU 上，NCCL 支持三种启动操作的执行模型

- 单 CPU 进程 - 单 GPU：将每个 GPU 绑定到不同 CPU 进程上，对应的进程可以被调度到固定 NUMA 节点（CPU - 内存 - GPU），提高数据局部性、减少内存访问延迟
- 单 CPU 线程 - 单 GPU：一个进程通过多个线程管理多个 GPU，所有线程可以访问共享内存（包括 GPU buffer 的 CPU 侧地址），减少数据拷贝
- 单 CPU 线程 - 多 GPU：一个线程管理多个 GPU，架构简单、CPU 开销小、执行确定性，可用于小型部署

### Communication Channel

NCCL 为了解决通信瓶颈问题，在设计上有几点：

- communication channel 调度：将大的集合通信操作分解成可以并行的 “子任务”，每个 channel 是处理子任务的独立执行单元
- 计算 / 通信重叠：为通信分配专门的 SM，避免 SM 同时负担计算、通信
- 数据并行：将输入缓冲分成多个 chunk，让多个 channel 可以并行处理
- 网卡均衡：将不同 channel 绑定到不同网卡，跑满物理链路带宽，避免单个 NIC 瓶颈

这里的 chunk size 具体取值也是并行 / 网络效率的 trade-off。如果每个 channel 的 chunk 太小、小于 NIC 的 512KB FIFO buffer size，那么网络带宽是部分浪费的。NCCL 有启发式算法，针对小消息减少 channel 数量来解决此问题。

NCCL 的内部调优模型会基于当前策略、message size、可用带宽、每个 channel 配置的 thread 数等因素，最终确定实际的 channel 数量。

分配给每个 channel 的逻辑通信拓扑直接决定了数据在 GPU 间的传输路径：

- 环状拓扑（ring）。每个 GPU 会找到前向、后继节点，所有 GPU 构成一个单向通信环
- 树状拓扑（tree）。每个 GPU 追踪父节点 / 子节点的 rank，构建一个通信树

为了增加带宽利用率，NCCL 使用双二叉树优化通信拓扑。
> 传统 AllReduce 的瓶颈
> - Ring AllReduce：带宽效率高，但是延迟 `O(n)` 高
> - Tree AllReduce：延迟 `O(log n)` 低，但是带宽利用率低（根节点成为瓶颈）
>
> NCCL 引入双二叉树，两棵树互为镜像：
> - Tree 0：以某节点为根，其他节点作为子树，数据向上 Reduce
> - Tree 1：以另一个（或相同）节点为根，结构对称，用于向下 Broadcast
>
> 下图中，左树的中间节点都是右树的叶子节点，右树的中间节点都是左树的叶子节点。AllReduce=Reduce+Broadcast，在 Reduce 操作中，叶子节点发数据，中间节点收数据，因此这样可以把带宽充分用起来
> ![alt text](img/image.png)
>

对于 `ncclGroupStart` 和 `ncclGroupEnd` 之间的点对点操作，NCCL 会尽可能让这些操作分散到多个 channel，让不依赖的 send/recv 并行。（task-level parallelism）

## Communication Protocols

三种通信协议 Simple、LL（Low Latency）、LL128，在带宽 / 延迟的 trade-off 不同：

![alt text](img/image-1.png)

### Simple Protocol

- 设计目标：充分利用带宽，用来传 large message
- 原理：chunking。将数据分成较大的 chunk 通过多个 channel 发送。

