# mmproxy

mmproxy 官方 repo 地址：<https://github.com/cloudflare/mmproxy>

mmproxy 是一个轻量级的 TCP 代理，让 TCP 服务器平滑地使用 proxy-protocol。

## 从 haproxy 说起

haproxy 是一个负载均衡器，主要用于 TCP 和 HTTP 应用。
它的特点：

- 高性能：可处理大量的并发连接
- 灵活性：支持多种负载均衡算法，比如轮询、最少连接等
- 健康检查：检测后端服务器的健康情况，以此分配流量
- 安全性：支持 SSL/TLS 加密等
- 可扩展：可通过配置文件进行定制和扩展

haproxy 的 proxy protocol：

proxy protocol 是 haproxy 为解决 “负载均衡器后端服务器获取 client 真实 IP” 而引入的一种协议。proxy protocol 在每个 TCP 连接的开始处插入一个报头，携带 client 的源 IP、目的 IP 信息，从而让后端服务器能获取这些信息。

proxy protocol 会传输 client 相关的信息，但是后端服务器需要支持和配置 proxy protocol 才能正确解析这些信息。像 nginx、apache 都是支持的。

但是，并不能保证所有的应用都能支持 proxy protocol，这时就需要 mmproxy 作为一个 workaround 了。

## mmproxy 工作原理

mmproxy 位于应用程序的附近工作，它接受来自负载均衡器的支持 proxy protocol 的连接，用伪装的 client IP 将流量直接发送给应用，因此在应用看来，流量是直接来自于 remote client 的。

- 伪装 IP：mmproxy 使用了 Linux 的 socket option `IP_TRANSPARENT`。这个选项允许用任意 IP 地址创建 socket。
- 将流量发送给应用：默认情况下，Linux 会将 response packet 发送到默认路由。mmproxy 会维护一个 custom routing table，强制将应用的返回流量发送到 loopback，之后 mmproxy 会捕获该流量并转发给 client

## go-mmproxy

go-mmproxy 的 repo 地址：<https://github.com/path-network/go-mmproxy>

go-mmproxy 是 go 版本实现的 mmproxy。和原版相比，go-mmproxy 据说运行更稳定，性能也更高。

## Troubleshoot

我在部署 haproxy-mmproxy 的时候，遇到了无法连接的情况，在部署 haproxy 的机器的 `/var/log/haproxy/traffic.log` 中有日志：

```log
# 异常
May 30 16:47:26 localhost.localdomain haproxy[5767]: 202.120.3.36:62358 [30/May/2024:16:47:19.349] ssh ssh/aaa 1/0/7102 0 SD 417/417/416/1/0 0/0
May 30 16:48:05 localhost.localdomain haproxy[5767]: 202.120.3.36:62374 [30/May/2024:16:47:58.257] ssh ssh/aaa 1/0/7106 0 SD 416/416/415/1/0 0/0

# 正常
May 30 17:10:38 localhost.localdomain haproxy[5767]: 10.184.5.189:39130 [30/May/2024:17:10:32.998] ssh ssh/bbb 1/0/5014 3489 -- 419/419/418/125/0 0/0
May 30 17:10:58 localhost.localdomain haproxy[5767]: 202.120.23.8:52588 [30/May/2024:16:10:44.271] ssh ssh/ccc 1/5/3613990 73277 cD 420/420/419/145/0 0/0
May 30 17:11:02 localhost.localdomain haproxy[5767]: 202.120.3.245:59303 [30/May/2024:17:10:53.863] ssh ssh/ccc 1/0/8366 26148145 -- 420/420/419/144/0 0/0
May 30 17:11:02 localhost.localdomain haproxy[5767]: 10.184.5.189:44772 [30/May/2024:17:11:00.390] ssh ssh/bbb 1/0/2300 3489 -- 419/419/418/125/0 0/0
May 30 17:11:07 localhost.localdomain haproxy[5767]: 10.148.20.119:63641 [30/May/2024:15:49:06.076] ssh ssh/ccc 1/0/4920991 180445 cD 419/419/418/143/0 0/0
May 30 17:11:14 localhost.localdomain haproxy[5767]: 202.120.58.243:47846 [30/May/2024:17:11:13.561] ssh ssh/bbb 1/0/949 3521 -- 420/420/419/125/0 0/0
May 30 17:11:34 localhost.localdomain haproxy[5767]: 10.184.5.189:41656 [30/May/2024:17:11:32.696] ssh ssh/ccc 1/0/2200 3489 -- 420/420/419/149/0 0/0
May 30 17:12:03 localhost.localdomain haproxy[5767]: 193.201.9.156:11216 [30/May/2024:17:11:59.602] ssh ssh/ccc 1/0/4265 1781 CD 422/422/421/149/0 0/0
May 30 17:12:15 localhost.localdomain haproxy[5767]: 139.227.141.74:20586 [30/May/2024:17:11:07.129] ssh ssh/ccc 1/4/68601 7841 -- 421/421/420/144/0 0/0
May 30 17:12:20 localhost.localdomain haproxy[5767]: 202.120.58.243:54994 [30/May/2024:17:12:18.641] ssh ssh/ccc 1/4/2178 3553 -- 421/421/420/144/0 0/0
May 30 17:12:34 localhost.localdomain haproxy[5767]: 10.181.192.187:62195 [30/May/2024:16:12:32.533] ssh ssh/bbb 1/0/3601737 4997 cD 420/420/419/125/0 0/0
May 30 17:12:43 localhost.localdomain haproxy[5767]: 58.246.235.2:24440 [30/May/2024:14:18:43.511] ssh ssh/bbb 1/0/10440185 19233 cD 420/420/419/124/0 0/0
```

解决方法：

按照 go-mmproxy 官方 repo，设置路由，让来自 loopback 的流量仍然回到 loopback：

（我只用了前两行）

```bash
ip rule add from 127.0.0.1/8 iif lo table 123
ip route add local 0.0.0.0/0 dev lo table 123

ip -6 rule add from ::1/128 iif lo table 123
ip -6 route add local ::/0 dev lo table 123
```

## 参考资料

- Cloudflare 关于 mmproxy 的 post：<https://blog.cloudflare.com/mmproxy-creative-way-of-preserving-client-ips-in-spectrum/>
