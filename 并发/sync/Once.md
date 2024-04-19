# 1 详细讲解下 Once

在 Go 语言中，`sync.Once` 是一个同步原语，它确保一个函数在多个 goroutine 中只被执行一次。这种机制非常有用，特别是在初始化共享资源或执行只需要一次的设置代码时。

### 如何工作

`sync.Once` 有一个重要的方法：`Do(f func())`。这个方法接收一个不接受参数也不返回任何值的函数 `f` 作为参数。无论多少次调用 `Once.Do(f)`，函数 `f` 只会被执行一次。这里的“一次”是指在程序的运行过程中，跨多个 goroutine 也是如此。

### 核心特性

-   **线程安全**：`sync.Once` 内部处理了所有必要的锁定机制，以保证即使在多个 goroutine 并发调用的情况下，初始化函数 `f` 也只被执行一次。
-   **效率**：一旦初始化函数被执行，`sync.Once` 会记录下这个事件。后续的 `Do` 调用将不会执行函数 `f`，而是直接返回。

### 示例

让我们通过一个例子来展示 `sync.Once` 的用法。假设有一个应用需要初始化一个配置对象，这个初始化过程必须且只能执行一次，即使在并发的环境下也是如此：

```go
package main

import (
    "fmt"
    "sync"
)

type Config struct {
    // 假设这里有很多配置字段
}

var (
    once     sync.Once
    instance *Config
)

// InitializeConfig 初始化全局配置
// 这个函数可以被多次调用，但实际的初始化代码只会执行一次
func InitializeConfig() *Config {
    once.Do(func() {
        // 模拟初始化过程
        instance = &Config{}
        fmt.Println("Config initialized")
    })
    return instance
}

func main() {
    var wg sync.WaitGroup
    // 模拟多个 goroutine 同时初始化配置
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            _ = InitializeConfig()
        }()
    }
    wg.Wait()
    fmt.Println("All goroutines finished")
}
```

### 示例说明

-   **Config 结构**：代表一个配置对象，这里没有具体的字段，只是作为示例。
-   **once**：`sync.Once` 类型的变量，用来确保配置的初始化代码只执行一次。
-   **InitializeConfig**：这个函数封装了 `once.Do` 调用，确保无论调用多少次，配置的初始化逻辑只会执行一次。每次调用都返回相同的配置实例。

### 输出

输出将显示 "Config initialized" 一次，即使有多个 goroutine 尝试执行初始化代码。在所有 goroutine 完成后，会打印 "All goroutines finished"。

可以看到，即使有 100 个协程都尝试实例化一个 Config 对象，但是字符串“实例化 Config”仅会打印一次，这代表调用 once.Do(func() {...}) 执行的匿名函数只被执行了一次，而通过匿名函数初始化的变量 instance 也只会一次性赋值，从而实现了单例模式。

### 使用场景

`sync.Once` 是初始化共享资源的理想选择，特别是在并发环境下需要延迟初始化的资源。它确保初始化的代码在多个 goroutine 中安全执行，而且效率高，不会因为频繁的锁操作而降低性能。
