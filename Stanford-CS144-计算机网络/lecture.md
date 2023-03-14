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



## 2-1 tcp service model

TCP工作在传输层

![image-20230312214011617](assets/image-20230312214011617.png)

TCP提供了端到端的通信，称为连接。

在连接的两端，TCP使用一个状态机来跟踪连接的状态

![image-20230312214136258](assets/image-20230312214136258.png)

tcp service model:

| property           | behaviour                                                    |
| ------------------ | ------------------------------------------------------------ |
| stream of bytes    | reliable byte delivery service                               |
| reliable delivery  | ack; checksum; sequence number; flow-control (avoid sender sends too fast) |
| in-sequence        | deliver in sequence                                          |
| congestion control | control network congestion                                   |

ISN: initial sequence number



## 2-2 udp service model

UDP数据报结构：

* 发送端口、接收端口
* length：整个UDP数据报的长度，头部+数据部分
* checksum：使用ipv4时，checksum是可选的。

![image-20230312220527247](assets/image-20230312220527247.png)

udp service model properties：

| property                        | behaviour                                                    |
| ------------------------------- | ------------------------------------------------------------ |
| connectionless datagram service | no connection; packets may show up in any order              |
| self contained datagrams        |                                                              |
| unreliable delivery             | no ack; no mechanism to detect missing or mis-sequenced datagrams; no flow control |



使用UDP的：DNS、DHCP



## 2-3 icmp service model

ICMP: internet control message protocol

工作在网络层上

两个属性：type和code

icmp service model:

| property          | behaviour                              |
| ----------------- | -------------------------------------- |
| reporting message | self-contained message reporting error |
| unreliable        | simple datagram service -- no retry    |

用例：ping、traceroute



## 2-4 end to end principle

端到端的文件传输的正确性：

* 网络可以提供帮助，但是不能对正确性负责
* 要保证端到端的正确性，只能让应用本身负责

例如，D在收到文件之后，保存再内存中的文件发生了比特翻转，D将错误的文件往后面传输了，由于网络只能检测传输中的错误，存储中发生的错误不能被发现

![image-20230312231911347](assets/image-20230312231911347.png)



## 2-5 error detection

* checksum
  * pro: fast and cheap
  * con: not very robust
* CRC, cyclic redundancy code
  * pro: stronger guarantee than checksum
  * con: more expensive than checksum
* MAC, message authentication code
  * pro: robust to malicious modifications
  * con: not robust to errors（和CRC相比）

MAC：

c=MAC(M, s)，c是生成的code，M是消息，s是secret值

链路层一般使用CRC



## 2-6 finite state machines

FSM of TCP:

![image-20230313154310358](assets/image-20230313154310358.png)

左下角的FIN_WAIT1是主动关闭连接的一方在发送了FIN之后进入的状态，它有三种可转移的状态：

* 正常情况，对方还有数据没发完，对方发了ACK，主动关闭方收到ACK之后，变成FIN_WAIT2
* 连接双方同时想主动关闭连接，对方也发了FIN，这时状态变成CLOSING
* 对方没有数据要发送了，对方发了FIN+ACK，这时状态变成TIME_WAIT



## 2-7 stop and wait

flow control:

* send <= recv
* two basic approaches: 
  * stop and wait
  * sliding window



stop and wait：

* one packet in flight at most
* sender sends one packet => recver recvs data and sends ack => sender sends new packet
* sender resends packet on timeout
* use 1-bit counter to detect duplication（让sender知道是重新发，还是发送新的包）



## 2-8 sliding window

sliding window:

* bound on number of un-acked segments, called window



sender maintain 3 variables:

* send window size (SWS)
* last ack recved (LAR)
* last segment sent (LSS)





recver maintain:

* recv window size (RWS)
* last acceptable segment (LAS)
* last segment recved (LSR)

LAS-lSR<=RWS

累计确认：收到1，2，3，5，需要确认3

ack：TCP ack是希望收到的下一个序号，例如要确认3，ack是4



## 2-9 reliable communication: retransmission strategy

* go back N，从丢失的包开始，都需要重传
* selective repeat，只传输丢失的包



## 2-10 reliable communication: tcp header

tcp header:

<img src="assets/image-20230314171242909.png" alt="image-20230314171242909" style="zoom:50%;" />

checksum: 包括pseudo header、TCP header、TCP data



## 2-11 reliable communication: tcp setup and teardown

tcp建立连接支持simultaneous open

tcp断开连接支持simultaneous close

![image-20230314175614009](assets/image-20230314175614009.png)



## 2-12 transport recap

使用UDP的应用程序：

* 应用有自己的私有方式处理重传
* 应用不需要可靠的交付



## 3-0 packet switching





