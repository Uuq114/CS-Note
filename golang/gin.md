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
    })
})
```

### 获取 postform 参数

通过 `DefaultPostForm` 和 `PostForm` 获取通过 form 表单提交的参数

```go
r.POST("/search", func(c *gin.Context) {
    username := c.DefaultPostForm("username", "lisi")
    city := c.PostForm("address")
    c.JSON(http.StatusOK, gin.H{
        "username": username,
        "city":     city,
        "status":   "ok",
    })
})
```

### 获取 json 参数

解析通过 json 提交的数据

```go
func jsonFunc(c *gin.Context) {
    b, _ := c.GetRawData()
    var m map[string]interface{}
    _ = json.Unmarshal(b, &m)
    c.JSON(http.StatusOK, m)
}
```

### 参数绑定

`ShouldBind` 可以基于请求自动提取 json、form 表单、QuertString 类型的数据，并把值绑定到指定的 struct 对象

`ShouldBind` 绑定数据的顺序：

* 如果是 GET 请求，只用 form-data 绑定
* 如果是 POST 请求，首先检查 Content-Type 是否为 json 或者 xml，再使用 form

```go
router.GET("/loginForm", func(c *gin.Context) {
    var login Login
    // ShouldBind()会根据请求的Content-Type自行选择绑定器
    if err := c.ShouldBind(&login); err == nil {
        c.JSON(http.StatusOK, gin.H{
            "user":     login.User,
            "password": login.Password,
        })
        } else {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        }
    })
```

## Gin 中间件

Gin 框架允许在处理请请求的过程中，加入用户自己的 hook 函数，这个 hook 函数就叫中间件。中间件适合处理一些公共业务逻辑，比如登录认证、权限校验、数据分页、日志记录、耗时统计等

中间件部分待补充