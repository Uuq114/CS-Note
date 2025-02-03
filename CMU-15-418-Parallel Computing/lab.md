# Lab Notes

<!-- TOC -->

- [Lab Notes](#lab-notes)
    - [Assignment 1: Analyzing Parallel Program Performance on a Quad-Core CPU](#assignment-1-analyzing-parallel-program-performance-on-a-quad-core-cpu)
        - [Program 1](#program-1)

<!-- /TOC -->
![alt text](img/image-33.png)

确实是thread 1比较慢

换成round-robin方法之后，即将所有行划分成一些chunk，线程按照0-1-2这样的顺序来计算，speedup上去了

![alt text](img/image-34.png)
