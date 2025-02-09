# Lab Notes

<!-- TOC -->

- [Lab Notes](#lab-notes)
  - [Assignment 1: Performance Analysis on a Quad-Core CPU](#assignment-1-performance-analysis-on-a-quad-core-cpu)
    - [Program 1: Parallel Fractal Generation Using Threads (20 points)](#program-1-parallel-fractal-generation-using-threads-20-points)
    - [Program 2: Vectorizing Code Using SIMD Intrinsics (20 points)](#program-2-vectorizing-code-using-simd-intrinsics-20-points)

<!-- /TOC -->
<!-- /TOC -->

## Assignment 1: Performance Analysis on a Quad-Core CPU

### Program 1: Parallel Fractal Generation Using Threads (20 points)

![alt text](img/image-33.png)

确实是thread 1比较慢

换成round-robin方法之后，即将所有行划分成一些chunk，线程按照0-1-2这样的顺序来计算，speedup上去了

![alt text](img/image-34.png)

### Program 2: Vectorizing Code Using SIMD Intrinsics (20 points)

