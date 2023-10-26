# http

Golang 的官方 http 包提供了 HTTP client/server 的实现

## 直接发起 HTTP 请求，获取 response

可以发起 Get、Post、PostForm请求，最后要关闭 `resp`

```golang
resp, err := http.Get("https://www.baidu.com")

```

## Client

可以自定义 `http.Client`，再用 client 发起请求

`http.Client` 有一个 `Transport` 属性，是控制代理、TLS配置、keep-alive等配置的

Client 和 Transport 都是可以多个 goroutine 一起用的

```golang
tr := &http.Transport{
    MaxIdleConns:       10,
    IdleConnTimeout:    30 * time.Second,
    DisableCompression: true,
}
client := &http.Client{Transport: tr}
resp, err := client.Get("https://example.com")
```

## Server

### 开启 Server

```golang
http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})

http.ListenAndServe(":8080", nil)
```

`ListenAndServe` 接收一个地址和 `handler` 来启动 server，可以通过 `http.Handle` 和 `http.HandleFunc` 增加 server 的路由和对应 `handler`

### 自定义 Server

如果想控制更多的 server 选项，可以自定义一个 server

```golang
type FooHandler struct {
}

func (h FooHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "hello, foo")
}

func main() {
    s := &http.Server{
        Addr:           ":9090",
        Handler:        MyHandler{},
        ReadTimeout:    5 * time.Second,
        WriteTimeout:   5 * time.Second,
        MaxHeaderBytes: 1 << 20,
    }
    log.Fatal(s.ListenAndServe())
}
```
