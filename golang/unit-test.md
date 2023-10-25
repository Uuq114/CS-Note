# Unit Test

实现文件为 `calculator.go`，测试文件为 `calculator_test.go`，测试文件里面的函数名称为 `TestXXX`，使用 `go test` 运行测试，`-v` 选项展示详细信息。

## 子测试

测试函数 `TestXXX` 中需要测试多组测试数据时，用 `t.Run` 执行一组测试用例

```golang
func TestXXX(t *testing.T){
    t.Run("case1", func(t *testing.T){...})
    t.Run("case2", func(t *testing.T){...})
    t.Run("case3", func(t *testing.T){...})
}
```

## 表格驱动测试

把测试用例用表格 `in, out` 列出来，结合 `t.Run` 跑多个用例的测试

```golang
var flagtests = []struct {
    in  string
    out string
}{
    {"%a", "[%a]"},
    {"%-a", "[%-a]"},
    {"%+a", "[%+a]"},
    {"%#a", "[%#a]"},
    {"% a", "[% a]"},
    {"%0a", "[%0a]"},
    {"%1.2a", "[%1.2a]"},
    {"%-1.2a", "[%-1.2a]"},
    {"%+1.2a", "[%+1.2a]"},
    {"%-+1.2a", "[%+-1.2a]"},
    {"%-+1.2abc", "[%+-1.2a]bc"},
    {"%-1.2abc", "[%-1.2a]bc"},
}
func TestFlagParser(t *testing.T) {
    var flagprinter flagPrinter
        for _, tt := range flagtests {
            t.Run(tt.in, func(t *testing.T) {
                s := Sprintf(tt.in, &flagprinter)
                if s != tt.out {
                t.Errorf("got %q, want %q", s, tt.out)
            }
        })
    }
}
```

通常表格是匿名结构体 slice，形如 `[]struct{string...}`，再结合 `t.Run` 遍历表格测试

### 并行测试

添加 `t.Parallel()` 可以将表格驱动测试并行化

```golang
for _, tt := range tests {
    tt := tt  // 注意这里重新声明tt变量（避免多个goroutine中使用了相同的变量）
    t.Run(tt.name, func(t *testing.T) { // 使用t.Run()执行子测试
        t.Parallel()  // 将每个测试用例标记为能够彼此并行运行
        got := Split(tt.input, tt.sep)
        if !reflect.DeepEqual(got, tt.want) {
            t.Errorf("expected:%#v, got:%#v", tt.want, got)
        }
    })
    }
```

### 测试覆盖率

测试中至少被运行一次的代码占总代码的比例，`go test -cover`
