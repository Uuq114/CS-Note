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