[toc]

* cs144 lab website: https://cs144.github.io/
* my solution is in: https://github.com/Uuq114/sponge



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

要实现一个TCP recver，可以将输入的重复的、无序的`index,data`整理成有序的`data`