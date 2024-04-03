# 1 什么是主 goroutine

```go
package main

import (
    "fmt"
    "time"
)

// 定义一个函数
func say(s string) {
    for i := 0; i < 5; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Println(s)
    }
}

func main() {
    // 以 goroutine 的方式运行 say 函数
    go say("world")
    // 主 goroutine 中运行
    say("hello")
}
```

分析下这段代码

在 Go 程序中，主 goroutine 是指程序的入口点所在的 goroutine。在你的例子中，`main` 函数是整个程序的入口点，因此在 `main` 函数中执行的代码都是在主 goroutine 中运行的。当你用 `go` 关键字启动一个新的 goroutine 时，这个新的 goroutine 会与主 goroutine 并发执行。

在你的代码示例中：

```go
func main() {
    // 以 goroutine 的方式运行 say 函数
    go say("world")
    // 主 goroutine 中运行
    say("hello")
}
```

-   当程序开始运行时，`main` 函数在主 goroutine 中执行。
-   使用 `go say("world")` 语句，你启动了一个新的 goroutine 来执行 `say("world")`。这个新启动的 goroutine 将并发地执行，独立于主 goroutine。
-   同时，主 goroutine 继续向下执行，并开始执行 `say("hello")`。

在这个场景下，`main` 函数内部直接调用 `say("hello")` 的部分就是在主 goroutine 中执行的。主 goroutine 负责启动其他 goroutine，并且在所有非主 goroutine 完成执行后继续执行，直到 `main` 函数返回，此时程序结束。如果主 goroutine 结束了，但是其他 goroutine 还在运行，那么这些 goroutine 也会被强制结束。

# 2 如果不使用 go 这个关键字，那么上面的 main 函数还有没有主 goroutine

即使你不显式使用 `go` 关键字来启动新的 goroutine，你的 Go 程序仍然会在一个主 goroutine 中运行。主 goroutine 是指运行 `main` 函数的 goroutine。在 Go 程序中，不管是否显式地创建新的 goroutine，主 goroutine 总是存在的，因为它是程序入口点的执行环境。

当你的程序不使用 `go` 关键字时，它不会创建新的并发执行流，所有代码都将在主 goroutine 中顺序执行。但这并不意味着没有主 goroutine，只是在这种情况下，程序中只有一个执行流——主 goroutine。

在这个上下文中，主 goroutine 的概念仍然适用，它指的是程序的主执行线程。如果整个程序中只有主 goroutine，那么它将是程序唯一的执行流。所有的函数调用和操作都将在这个主 goroutine 中按顺序执行，直到程序结束。

# 3 go say("world") 会创建多个协程么

当你使用 `go say("world")` 这行代码时，你仅创建了一个新的 goroutine 来执行 `say` 函数。这行代码并不会创建很多个 goroutine；它只创建了一个单独的 goroutine 来并发地执行 `say` 函数。

每次你使用 `go` 关键字时，你就会创建一个新的 goroutine。如果你想创建多个 goroutine 来执行相同的函数或不同的函数，你需要为每个想要并发执行的函数调用前加上 `go` 关键字。例如：

```go
func main() {
    go say("world") // 创建第一个 goroutine 来执行 say 函数
    go say("hello") // 创建第二个 goroutine 来执行 say 函数
    // 如果需要，可以继续创建更多的 goroutine
}
```

在这个例子中，我们创建了两个 goroutine：一个执行 `say("world")`，另一个执行 `say("hello")`。这两个 goroutine 将与主 goroutine（执行 `main` 函数的那个）并发执行。

简而言之，每个使用 `go` 关键字的地方都会创建一个新的 goroutine，而 `go say("world")` 只创建了一个 goroutine 来执行 `say` 函数。

# 4 go say("world") 创建了一个 goroutine，那么是怎么控制函数内的代码并发执行的

当你使用 `go say("world")` 创建了一个 goroutine 时，Go 语言的运行时（runtime）负责调度这个新创建的 goroutine 以及其他存在的 goroutine（包括主 goroutine）。每个 goroutine 在内部是独立执行的，但它们是在 Go 的运行时环境中并发运行的。下面是这个过程的大致描述：

### 1. 创建 goroutine

当你在代码中使用 `go` 关键字 followed by 函数调用时，Go 运行时会创建一个新的 goroutine。这个 goroutine 会包含该函数调用的执行环境。在你的例子中，`go say("world")` 告诉 Go 运行时创建一个新的 goroutine 来执行 `say` 函数，并且传递 `"world"` 作为参数。

### 2. 调度和执行

创建了 goroutine 后，Go 的调度器会将其加入到等待运行的 goroutine 队列中。调度器负责管理所有 goroutine 的执行：

-   **调度策略**：Go 使用的是 M:N 调度模型，意味着它会将 M 个 goroutine 调度到 N 个操作系统线程上执行。调度器在运行时动态地管理这些 goroutine，使得它们在可用的线程上得到执行。
-   **并发执行**：虽然多个 goroutine 可能会并发执行，但在任何给定时刻，一个核心上只会执行一个 goroutine。如果有多个处理器核心可用，Go 运行时会利用这些核心来实现真正的并行执行。

### 3. 协作式抢占

Go 的 goroutine 使用的是协作式抢占。这意味着一个 goroutine 会继续执行，直到它自愿放弃执行权（比如通过进行 I/O 操作，调用 `time.Sleep`，或者显式的调用 `runtime.Gosched()` 来让出 CPU 时间）。

### 4. 结束执行

当 `say` 函数内的代码执行完毕后，这个 goroutine 就会自动结束。如果这是程序中唯一的或最后一个活跃的 goroutine，程序就会结束。如果还有其他 goroutine 在运行，Go 的运行时会继续调度和执行这些 goroutine。

在你的 `say` 函数例子中，函数内部通过 `time.Sleep` 在每次打印后暂停，这不仅模拟了延时操作，也为 Go 的调度器提供了调度其他 goroutine 的机会，从而实现并发执行。
