[toc]

* cs144 lab website: https://cs144.github.io/
* my solution is in: https://github.com/Uuq114/sponge



- [x] Lab 0: networking warmup
- [x] Lab 1: stitching substrings into a byte stream
- [x] Lab 2: the TCP receiver
- [ ] Lab 3: the TCP sender
- [ ] Lab 4: the TCP connection
- [ ] Lab 5: the network interface
- [ ] Lab 6: the IP router



## lab0

2.1

X-Your-Code-Is: 677018



2.3

netcat -v -l -p 9090：

```bash
# uuq114 @ LAPTOP-51P4L641 in ~ [14:02:29]
$ netcat -v -l -p 9090
Listening on [0.0.0.0] (family 0, port 9090)
Connection from localhost 4016 received!
123
^C
```



3.4

使用SHUT_WR关闭写，避免服务器等待

```c++
TCPSocket sock;
sock.connect(Address(host, "http"));
const string input(
    "GET " + path + " HTTP/1.1\r\n" +
    "Host: " + host + "\r\n" +
    "Connection: close" + "\r\n\r\n"
);
sock.write(input);
sock.shutdown(SHUT_WR);
while(!sock.eof()) {
    cout << sock.read();
}
sock.close();
```



4

`writer ====> buffer =====> reader`

deque模拟的buffer

第9个test，t_socket_dt没过，测试文件是doctests/socket_example_1、2、3

> 我用的wsl，可能和这个有关系，下次用virtualbox跑一遍



## lab1

https://cs144.github.io/assignments/lab1.pdf

要实现一个TCP recver，可以将输入的重复的、无序的`index,data`整理成有序的`data`



eof:

用户如果在push string的时候指定eof为true，代表该string的最后一个字母是流的结尾。因此使用eof_index存储该位置，当接收到大于eof_index的数据时，将其丢弃。



## lab2

要实现一个TCP recver，接收传入的TCP segment，组装byte stream，并且向发送者返回必要的信息（ack和窗口大小）



需要安装libpcap：

```bash
sudo apt-get install libpcap-dev
```



TCP报文头部中使用32位的数来表示报文的序号，有几个可能的问题：

* 序号一直递增，最后可能会溢出32位能表达的最大值
* 初始序号是随机的，所以最后消息的序号超过2^32时，要用取余的方法
* SYN和FIN也占用序号，它们不属于字节流的一部分



![image-20230317234110837](assets/image-20230317234110837.png)

unwrap思路：

开始将checkpoint转化成wrap_int32，通过和isn做差获得offset，在计算n时，让n尽可能从右边接近checkpoint。checkpoint是之前实现过的ByteStream中的第一个未组装的字节索引号（64位，从0开始）。unwrap返回的是最接近checkpoint的值。

当ret小于0时（这里ret声明的类型是有符号的int64），加上1<<32，对应这种情况：
``` 
   ======.==========.======.=========
  0      c         isn     n       1<<32
```

```c++
uint64_t unwrap(WrappingInt32 n, WrappingInt32 isn, uint64_t checkpoint) {
    WrappingInt32 c = wrap(checkpoint, isn);
    int32_t offset = n.raw_value() - c.raw_value();
    int64_t ret = checkpoint + offset;
    return ret >= 0 ? ret : ret + (1ul << 32);
}
```

> string_view:
> 跟std::string相比，在创建std::string_view对象的时候，没有任何动态内存分配，没有对字符串多余的遍历
>
> optional:
>
> 用来实现多个返回值的



## lab3

lab3要求实现一个TCPSender，它将字节流转化成TCP segment，发送给recver，如果没有收到ack，重新发送对应的包

TCPSender要写的部分有TCP segment：seqno、SYN、payload、FIN

TCPSender要读的部分是：ackno、window size

tick方法：会返回距离上次调用tick经过的时间，单位是ms

RTO：重传超时时间。这个值会变动，但是初值是确定的

timer：重传的时钟。可以在某个时间开始计时，在RTO到达时过期。在发送一个TCP segment时，要启动一个时钟。



有一个要注意的：

假设发送方发送的数据占满了接收方的缓冲区，接收方返回了一个ack，同时携带窗口大小为0。这时发送方停止发数据，只会在收到新的ack之后才会发数据，同时接收方又因为是携带确认而不会发送ack，整个系统会死锁。



## lab4

实现一个TCPConnection，将TCPSender和TCPRecver组合起来

TCPConnection相当于连接两端的endpoint，在其内部维护了一个TCPSender和一个TCPRecver

**TCPConnection在接收segment时：**

* 如果flag=RST，关闭连接
* 将segment转交给TCPRecver
* 如果flag=ACK，告诉TCPSender：ackno、window size
* 如果incoming segment使用了序列号，TCPConnection需要回复ack和window size
* 处理keepalive segment：探活segment的序号是invalid的，需要回复对方current window

**TCPConnection在发送segment时：**

* TCPSender向发送队列添加了segment之后，TCPConnection需要发送这个segment
* 在发送segment之前，TCPConnection会向TCPRecver询问ackno和window size。如果有ackno，需要设置segment的ack flag。

**随时间变化：**

TCPConnection的tick方法会被OS周期调用，tick调用的时候TCPConnection需要做：

* 告诉TCPSender已经流逝的时间
* 如果重传次数超过了最大值，向peer发送一个reset segment（RST flag，空内容）
* 可能需要关闭连接

**连接关闭：**

* 非正常的关闭：通过RST，此时TCPConnection的ByteStream要设置error，active状态设置为false
* 正常关闭：



参考：

https://deepz.cc/2021/08/cs144-lab4/

https://gwzlchn.github.io/202205/cs144-lab4/#%E8%B0%83%E8%AF%95%E5%BF%83%E5%BE%97



