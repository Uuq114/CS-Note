# Triton

## 概述

triton 特点：

- 既是基于 python 的 DSL（语言），也是编译器
- 面向 GPU 体系特点，自动分析和实施神经网络计算的分块（tiled neural network compute）
- 和 CUDA 相比，Triton 提供更易用的编程模型；和 TVM/XLA 等图级编译器相比，Triton 用于 kernel 开发和编译优化

kernel 算子开发：大量算子（linear/convolution/norm/pooling/loss）和多种数据类型（f64/f32/bf16/int64/int32）的不同组合

![alt text](img/image.png)

triton 编译器：

- 兼容 pytorch 等框架
- 基于 MLIR 的多层中间表示和优化
  - Triton dialect
  - TritonGPU dialect
- 利用 LLVM 生成不同硬件平台的高效执行代码

![alt text](img/image-1.png)

## Triton API 语法

`jit`: 即时编译

`autotune`: 对 `triton.jit` 函数自动调优，比如 `num_warps`、`num_stages`、block size 等

![alt text](img/image-2.png)

debug ops，调试使用的函数，不能用 python 的 `print()`

![alt text](img/image-3.png)