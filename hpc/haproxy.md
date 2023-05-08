# HAProxy

[toc]

## load balancer

load balancer 起作用的位置可能有：

* link level：有多个 network link，每个 packet 要选择一个 link
* network level：有多个 route，一串 packet 要选择一个 route
* server level：有多个 server，connection/request 要选择一个 server



```
        =========================
A ======||    load balancer    ||====== B
        =========================
```

load balancer 主要有两种实现方式：

|           | packet-based                                    | proxy-based                                                  |
| --------- | ----------------------------------------------- | ------------------------------------------------------------ |
| 操作粒度  | packet                                          | session content                                              |
| 输入-输出 | 输入的 packet 和输出的 packet 是对应的关系      | 输入的流会被重新组装，内容会被改变，输出的流是由新的 packet 组成 |
| 效率      | 高                                              | 低，尤其是处理 small packet 时                               |
| 响应      | B 的响应可以直接不经过 LB 返回给 A              | B 的响应必须经过 LB 返回                                     |
| 适合场景  | network-level LB                                | server-level LB                                              |
| 部署      | 一般就部署在正常的网络传输路径上（cut-through） | 一般有自己的 IP 和 port，                                    |



## HAProxy

high availability proxy

http://docs.haproxy.org/2.7/intro.html#2

高可用的代理软件，可以做四层或者七层的代理

事件驱动，单一进程模型



## haproxy和nginx的对比

