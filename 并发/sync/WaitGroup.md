# 1 WaitGroup 基础讲解

`sync.WaitGroup` 是 Go 语言标准库中的一个结构，用于等待一组 goroutines 的执行完成。它非常适合处理并发任务时，当你需要等待所有任务完全完成后再继续执行。`WaitGroup` 主要提供了三个方法：`Add(delta int)`, `Done()`, 和 `Wait()`。

### WaitGroup 的核心方法

1. **Add(delta int)**: 此方法用于设置或增加等待组中的计数器。通常在启动 goroutine 之前调用，设置需要等待完成的 goroutine 数量。
2. **Done()**: 每个 goroutine 完成其任务后应调用此方法。它将 `WaitGroup` 的内部计数器减一。
3. **Wait()**: 此方法会阻塞调用它的 goroutine 直到计数器减到零，即所有的 goroutine 都调用了 `Done()` 方法。

### 示例代码

下面的示例代码将展示如何使用 `WaitGroup` 来同步多个并发执行的任务：

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var wg sync.WaitGroup
	workerCount := 5

	for i := 0; i < workerCount; i++ {
		wg.Add(1) // 为每个 goroutine 增加等待计数
		go func(id int) {
			defer wg.Done() // 确保 goroutine 完成后递减 WaitGroup 的计数
			work(id)
		}(i)
	}

	wg.Wait() // 等待所有 goroutine 完成
	fmt.Println("All tasks completed.")
}

func work(id int) {
	fmt.Printf("Worker %d starting\n", id)
	time.Sleep(time.Second * 2) // 模拟耗时任务
	fmt.Printf("Worker %d done\n", id)
}
```

### 代码说明

-   **设置 WaitGroup**: 在启动每个 goroutine 之前，我们调用 `wg.Add(1)` 来告知 `WaitGroup` 需要等待一个额外的任务完成。
-   **启动 Goroutines**: 启动多个 goroutine 来执行 `work` 函数，每个 goroutine 代表一个工作任务。传递的 `id` 用于标识不同的工作者。
-   **结束通知**: 每个 goroutine 在完成其任务后通过调用 `defer wg.Done()` 来通知 `WaitGroup` 它已经完成了。这里使用 `defer` 确保即使函数中发生错误也会调用 `Done()`。
-   **等待完成**: 主 goroutine 调用 `wg.Wait()` 来阻塞，直到所有由 `WaitGroup` 跟踪的 goroutine 都调用了 `Done()`，即所有任务都完成了。

### 使用场景

`WaitGroup` 在并发编程中非常有用，特别是当你需要启动多个任务并等待它们全部完成后才能继续执行程序的下一部分。它简化了并发同步和状态管理，避免了复杂的通道（channel）操作或其他同步技术。这使得代码更简洁、更易于维护。
