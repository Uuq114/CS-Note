[toc]

* cs144 lab website: https://cs144.github.io/
* my solution is in: https://github.com/Uuq114/sponge



- [x] Lab 0: networking warmup
- [x] Lab 1: stitching substrings into a byte stream
- [ ] Lab 2: the TCP receiver
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