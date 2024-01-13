# Lustre OPA 笔记

<!-- TOC -->

- [Lustre OPA 笔记](#lustre-opa-笔记)
  - [Intel Omni-Path Architecture](#intel-omni-path-architecture)
  - [High Speed Networking Monitoring Enhancements](#high-speed-networking-monitoring-enhancements)

<!-- /TOC -->

## Intel Omni-Path Architecture

Intel官方出品的文章。
Intel OPA位于OSI的1-2层，每个Node通过Host Fabric Interface(HFI)连接到switch，类似于：Node=>HFI=>switch。OPA的管理者称为fabric manager，可以有后备管理者，管理者需要监控网络拓扑、网络性能（error rate等）。

## High Speed Networking Monitoring Enhancements

Los Alamos实验室介绍的slides。有一个开源项目HSNmon：<https://github.com/lanl/hsnmon>

目标：

- 针对OPA网络，频繁（~10s），收集switch、NIC上的port-level traffic bandwidth
- 收集switch的转发表

HSNmon介绍：

- 运行在fabric manager node上，可以导出error/performance counter数据
- 收集数据的方法：FastFabric tool，opaextractperf

挑战：

- 针对HSNmon运行较慢，主进程每次查询会拉起单独的进程
- 针对FastFabric tool（opaextractperf）运行较慢，HSNmon使用OPAMGT API来查询
