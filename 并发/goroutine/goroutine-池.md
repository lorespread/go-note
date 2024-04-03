# 1 详细讲解下 goroutine 池和对应的使用示例

在 Go 语言中，goroutine 池（goroutine pool）是一种用于管理并限制正在执行的 goroutine 数量的模式。虽然创建 goroutine 的成本很低，但是过多的 goroutine 可能会导致资源竞争和调度开销，因此在一些情况下，使用 goroutine 池来重用一组固定数量的 goroutine 可以更有效。

Go 语言标准库中没有直接提供 goroutine 池的实现，但是你可以使用 channel 和 sync 包来自己实现一个简单的 goroutine 池。

### 基本思想

goroutine 池通常由以下几个组件组成：

-   一个任务队列，用于存放待处理的任务。
-   一组工作 goroutine，从任务队列中获取任务并执行。
-   一个控制器，用于控制工作 goroutine 的数量和分配任务。

### 使用示例

下面是一个简单的 goroutine 池的示例实现：

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// Task 是我们要执行的工作
type Task func()

// Pool 是我们的 goroutine 池
type Pool struct {
	tasks         chan Task
	concurrency   int
	wg            sync.WaitGroup
}

// NewPool 创建一个新的 goroutine 池
func NewPool(concurrency int) *Pool {
	return &Pool{
		tasks:       make(chan Task),
		concurrency: concurrency,
	}
}

// Run 启动 goroutine 池
func (p *Pool) Run() {
	for i := 0; i < p.concurrency; i++ {
		p.wg.Add(1)
		go p.worker()
	}
}

// Shutdown 等待所有任务完成并停止 goroutine 池
func (p *Pool) Shutdown() {
	close(p.tasks)
	p.wg.Wait()
}

// AddTask 添加任务到 goroutine 池
func (p *Pool) AddTask(task Task) {
	p.tasks <- task
}

// worker 是执行任务的 goroutine
func (p *Pool) worker() {
	defer p.wg.Done()
	for task := range p.tasks {
		task()
	}
}

func main() {
	pool := NewPool(3) // 创建一个有 3 个工作 goroutine 的池
	pool.Run()

	// 添加一些任务到 goroutine 池
	for i := 0; i < 10; i++ {
		count := i
		pool.AddTask(func() {
			fmt.Printf("Processing task %d\n", count)
			time.Sleep(1 * time.Second)
		})
	}

	pool.Shutdown() // 关闭 goroutine 池并等待所有任务完成
	fmt.Println("All tasks completed.")
}
```

在这个例子中：

-   `Task` 类型是一个没有参数和返回值的函数类型，用于表示工作任务。
-   `Pool` 结构体表示 goroutine 池，它有一个 `tasks` channel 用于接收任务，`concurrency` 表示池中 goroutine 的数量，和 `wg` 是 sync.WaitGroup 的实例，用于等待所有 goroutine 完成。
-   `NewPool` 函数创建一个新的 goroutine 池。
-   `Run` 方法启动指定数量的 worker goroutines，它们从 tasks channel 中接收并执行任务。
-   `Shutdown` 方法关闭 tasks channel 并等待所有的 worker goroutines 停止。
-   `AddTask` 方法向 goroutine 池的任务 channel 中添加一个新任务。
-   `worker` 方法是在每个 worker goroutine 中运行的，它从 tasks channel 中接收任务并执行它们。

这个简单的 goroutine 池允许你控制 goroutine 的并发数量，以避免开销过大。通过关闭 tasks channel 和使用 sync.WaitGroup，它确保在程序退出前所有的任务都被处理。这只是一个基础的实现，实际的 goroutine 池可能还会包括错误处理、任务结果返回等功能。
