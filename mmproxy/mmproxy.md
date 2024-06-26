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

## 参考资料

- Cloudflare 关于 mmproxy 的 post：<https://blog.cloudflare.com/mmproxy-creative-way-of-preserving-client-ips-in-spectrum/>
