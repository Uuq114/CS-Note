# Lab Notes

<!-- TOC -->

- [Lab Notes](#lab-notes)
  - [Assignment 1: Performance Analysis on a Quad-Core CPU](#assignment-1-performance-analysis-on-a-quad-core-cpu)
    - [Program 1: Parallel Fractal Generation Using Threads (20 points)](#program-1-parallel-fractal-generation-using-threads-20-points)
    - [Program 2: Vectorizing Code Using SIMD Intrinsics (20 points)](#program-2-vectorizing-code-using-simd-intrinsics-20-points)

<!-- /TOC -->

## Assignment 1: Performance Analysis on a Quad-Core CPU

### Program 1: Parallel Fractal Generation Using Threads (20 points)

![alt text](img/image-33.png)

确实是 thread 1 比较慢

换成 round-robin 方法之后，即将所有行划分成一些 chunk，线程按照 0-1-2 这样的顺序来计算，speedup 上去了

![alt text](img/image-34.png)

### Program 2: Vectorizing Code Using SIMD Intrinsics (20 points)

1. 见代码
2. vector width 为 2、4、8、16 时的 vector utilization 如下图：
![alt text](img/image-119.png)
vector utilization 表示 enabled vector lane 的比例，因此 exponent 的分布会影响该值。当 vector width 越小时，对 exp 分的组越小，极小值的影响也小。
3. 见代码。先分组计算 vector，再对 vector 内求和。
