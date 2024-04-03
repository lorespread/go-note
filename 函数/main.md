在 Go 语言中，`main` 函数是一个特殊的函数，它作为程序的入口点。每个可执行的 Go 程序都必须包含一个 `main` 函数，这个函数位于 `main` 包中。`main` 函数不接受任何参数，也不返回任何值。当 Go 程序启动时，`main` 函数被自动调用，程序的执行从这里开始。

### 特点

-   **位置**：`main` 函数必须位于 `main` 包中。这意味着你的程序入口文件的包声明部分应该是 `package main`。
-   **签名**：`main` 函数没有参数也不返回值。其正确的声明形式是 `func main()`。
-   **自动执行**：当程序启动时，`main` 函数自动被调用。你无需手动调用它。
-   **程序结束**：`main` 函数的返回表示程序的结束。如果你需要在程序退出时返回一个状态码，应该使用 `os.Exit(statusCode int)` 函数。
-   **唯一性**：每个可执行的 Go 程序中只能有一个 `main` 函数。

### 使用示例

**示例 1：一个简单的 Hello World 程序**

这是最基本的 Go 程序，它在 `main` 函数中打印出 "Hello, World!"。

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

**示例 2：处理命令行参数**

Go 程序可以通过 `os.Args` 来接收和处理命令行参数。`os.Args` 是一个字符串切片（slice），包含了程序启动时传递给它的所有命令行参数。`os.Args[0]` 是程序的名称，后续元素是传递给程序的参数。

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // 打印所有命令行参数
    fmt.Println("Command-line arguments:")
    for index, arg := range os.Args {
        fmt.Printf("arg[%d] = %s\n", index, arg)
    }
}
```

如果你保存上面的代码为 `main.go` 并使用 `go run main.go arg1 arg2 arg3` 运行，它会打印出程序名称和传递给程序的三个参数。

### 注意事项

-   **初始化顺序**：在 `main` 函数执行之前，所有被导入的包的 `init` 函数（如果存在）会按照包导入的依赖顺序先后自动执行。
-   **并发**：如果你的程序需要并发执行任务，你可以在 `main` 函数中启动协程（goroutines）。程序会在 `main` 函数返回后结束，所以如果有需要，你应该使用如 `sync.WaitGroup` 或其他同步机制来等待协程完成。
-   **退出状态码**：默认情况下，如果 `main` 函数正常返回，程序的退出状态码是 0。如果需要返回非 0 状态码，可以使用 `os.Exit` 函数，并传递一个整数状态码。

`main` 函数是 Go 程序的起点，理解它的作用和如何使用是学习 Go 语言的基础部分。
