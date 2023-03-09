[toc]

## 简介

* 官网，包括lecture和lab：https://cs144.github.io/
* slides: https://github.com/khanhnamle1994/computer-networking



## 1-1

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



## 1-2

四层网络模型：application、transport、network、link

internet protocol (IP)：

* 尽力交付
* IP datagram可能丢失、乱序送达、损坏



## 1-3

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