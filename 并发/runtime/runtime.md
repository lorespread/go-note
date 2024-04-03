# 1 详细讲解下 runtime

Go 的 `runtime` 包提供了与 Go 运行时系统交互的功能，允许程序在运行时进行自管理。`runtime` 包提供的功能包括：goroutine 的管理，垃圾回收控制，内存分配，处理器数量设置，以及获取 goroutine 的堆栈信息等。下面我们将探讨 `runtime` 包的几个关键功能及其示例：

### 1. 查询和设置 GOMAXPROCS

`GOMAXPROCS` 设置决定了程序执行时有多少个 CPU 核心可用，它直接影响到并发性能。你可以使用 `runtime.GOMAXPROCS` 函数来查询或设置这个值。

```go
package main

import (
    "fmt"
    "runtime"
)

func main() {
    fmt.Println("Current GOMAXPROCS:", runtime.GOMAXPROCS(0)) // 查询当前 GOMAXPROCS 值

    runtime.GOMAXPROCS(4) // 尝试设置 GOMAXPROCS 为 4
    fmt.Println("New GOMAXPROCS:", runtime.GOMAXPROCS(0)) // 查询设置后的 GOMAXPROCS 值
}
```

### 2. 启动和查询 Goroutine

虽然 `runtime` 包本身不直接用于启动 goroutines，但可以与 goroutine 的调度和管理相关联，如查询当前存在的 goroutine 数量。

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func printNumbers() {
    for i := 0; i < 5; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Println(i)
    }
}

func main() {
    go printNumbers()

    fmt.Println("Number of goroutines:", runtime.NumGoroutine())

    time.Sleep(1 * time.Second) // 等待足够长的时间以确保 goroutine 执行完成
}
```

### 3. Goroutine 的堆栈信息

`runtime` 包允许你打印当前所有 goroutine 的堆栈信息，这在调试死锁或者分析并发行为时非常有用。

```go
package main

import (
    "runtime"
    "time"
)

func main() {
    go func() {
        time.Sleep(2 * time.Second) // 模拟一个耗时操作
    }()

    time.Sleep(1 * time.Second) // 确保上面的 goroutine 已经启动

    buf := make([]byte, 1<<20) // 分配足够大的 buffer
    n := runtime.Stack(buf, true) // 获取所有 goroutine 的堆栈信息
    println(string(buf[:n]))
}
```

### 4. 垃圾回收

`runtime` 包允许程序控制垃圾回收器的运行，例如，你可以手动触发垃圾回收：

```go
package main

import (
    "runtime"
)

func main() {
    runtime.GC() // 手动触发垃圾回收
}
```

虽然在实际应用中，手动触发垃圾回收很少见，因为 Go 的垃圾回收器设计得足够智能，能够根据当前内存使用情况自动运行。

`runtime` 包还提供了更多的功能，例如处理程序的内存分配、检测并处理程序的死锁情况等。这些工具对于深入理解和优化 Go 程序的性能非常有用。
