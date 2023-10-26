# Gin

Gin 是一个 Go 编写的 Web 框架。和 `martini` 相比，在性能上有较大的优势。

## Gin 初体验

下载安装 Gin：`go get -u github.com/gin-gonic/gin`

```golang
func main() {
    // 创建一个默认的路由引擎
    r := gin.Default()
    // GET：请求方式；/hello：请求的路径
    // 当客户端以GET方法请求/hello路径时，会执行后面的匿名函数
    r.GET("/hello", func(c *gin.Context) {
        // c.JSON：返回JSON格式的数据
        c.JSON(200, gin.H{
            "message": "Hello world!",
        })
    })
    // 启动HTTP服务，默认在0.0.0.0:8080启动服务
    r.Run()
}
```

## Gin 渲染

* HTML 文件：Gin 使用 `LoadHTMLGlob()` 或者 `LoadHTMLFiles()` 进行 HTML 模板渲染
第一个是指定 pattern 的，第二个是指定文件名称的
* 静态文件：在渲染页面前，先指定静态文件路径

    ```golang
    r := gin.Default()
    r.Static("/static", "./static")
    ```

* 渲染返回的JSON：定义一个 struct，将其作为 json 返回，或者用 `gin.H` 返回（类似于 `map[string]interface{}`）

    ```golang
    // 1
    var msg struct {
        ...
    }
    c.JSON(http.StatusOK, msg)
    // 2
    c.JSON(http.StatusOK, gin.H{
        "message": "hello",
    })
    ```

* protobuf 渲染：通过 `c.ProtoBuf(http.StatusOK, data)` 返回

## 获取参数

### 获取 querystring 参数

`querystring` 指 URL 中 `?` 后面的参数

```golang
r.Get("/search", func(c *gin.Context) {
    username := c.DefaultQuery("username", "lisi")
    city := c.Query("address")
    c.JSON(http.StatusOK, gin.H{
        "username": username,
        "city":     city,
        "status":   "ok",
    }
})
```
