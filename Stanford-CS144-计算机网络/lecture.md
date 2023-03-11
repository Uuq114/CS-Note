[toc]

## 简介

* 官网，包括lecture和lab：https://cs144.github.io/
* slides: https://github.com/khanhnamle1994/computer-networking



## 1-1 a day in the life of an application

world wide web (http)：

* client-server
* request-response



BitTorrent：

<img src="assets/image-20230309231741873.png" alt="image-20230309231741873" style="zoom:50%;" />

* client从其他的client下载文件
* 每个文件被拆成了多个部分
* tracker是一个节点，它记录了哪些client是群（swarm）的成员
* 使用torrent下载时，client首先从tracker获取其他client的名单，再从这些client下载文件



skype：

 <img src="assets/image-20230309232541674.png" alt="image-20230309232541674" style="zoom:50%;" />

* 两个用户通信时，相当于在它们之间建立了通信
* 当client B这边存在NAT时，client  A是不能主动连接B的。这时需要一个在外部的集合服务器C，B和A可以主动连接C，C通知B有消息来自A，然后B再主动连接A
* 当两边都有NAT时，需要有一个中继服务器D，A和B都能主动连接D，D再中间起到转发的作用



## 1-2 the four layer internet model

四层网络模型：application、transport、network、link

internet protocol (IP)：

* 尽力交付
* IP datagram可能丢失、乱序送达、损坏



## 1-3 the ip service model

IP service model：

* datagram：每个包是单独路由的，从一个路由器到下一个路由器
* 不可靠的：包可能丢失，但是尽最大努力交付
* 无连接的：

IP对数据链路层的假设很少，这让IP层可以在热呢数据链路层上工作，有线的，无线的，甚至信鸽（IPOAC）！

IP service model的其他特点：

* 避免数据报在路由器环路中无限传输：使用TTL
* 如果数据长度太长，数据报会被拆分
* 使用校验和来验证数据报是否被送到了错误的地方
* ipv4，ipv6
* IP允许将新字段添加到header中



## 1-4 a day in the life of a packet

wireshark

traceroute



## 1-5 packet switching principle

packet：包含了让包能到达目的地的必要信息

packet switching：路由器对每个包，独立地选择这个包的出路

flow：属于同一个端到端的交流的数据报的集合，例如：TCP连接

路由器不需要保存per-flow state

packet switching的优点：

* 简单：每个包是独立处理的，不需要知道flow的状态
* 高效：在共享连接的多个流之间，有效地共享容量（capacity，带宽？）



## 1-6 layering principle

分层的优点：

* 模块化
* 定义明确的服务
* 重用
* 关注点分离：每一层只要关注自己的任务



## 1-7 encapsulation principle

Frame => IP => TCP => HTTP

![image-20230311224236932](assets/image-20230311224236932.png)

上图中，headers也可以在左边

在VPN中，网关通过转发IP层的包来实现访问内部网络：

<img src="assets/image-20230311225519585.png" alt="image-20230311225519585" style="zoom:50%;" />

Eth => IP(VPN) => TCP(VPN) => TLS(VPN) => IP => TCP => HTTP



## 1-8 byte order

little endian：低位低字节，高位高字节

big endian：高位低字节

例子：10进制的4116等于16进制的0x1014，这是big endian

网络字节序都是big endian

C语言提供了在网络字节序和主机字节序之间的转换函数：

* 处理short的：htons、ntohs
* 处理long的：htonl、ntohl



## 1-9 ipv4 address

子网掩码：判断两个ip是否在同一个网络

CIDR：管理和分配ipv4地址



## 1-10 longest prefix match

路由器在决定包的转发地址时，匹配原则是目标地址和下一跳地址有最长的前缀

路由器的转发表里面有一系列的CIDR地址



## 1-11 address resolution protocol (ARP)

ARP协议：已经知道了数据包的下一跳的地址，需要知道将其发送到哪个link address

link address：描述了特定的网卡，发送和接收信息的唯一设备



ARP解析过程：

* 每个节点都有一个IP地址到MAC地址的映射的缓存
* 如果目标节点不在缓存中，那么发送一个request到链路层广播地址
* 目标节点收到请求后，reply给发请求的节点；其余节点收到广播request后，会更新自己的缓存



## 1-12 the internet and ip recap

* how an application uses the internet
  * browser
  * bittorrent
  * skype
* the structure of the internet: 4 layer model
* the internet protocol
* basic architectural ideas and principles
  * packet switching
  * layering
  * encapsulation



