# 1 错误使用 1

```go
package main

import (
	"fmt"
	"time"
)

func OneGoRoutine() {
	fmt.Println("Hello Goroutine")
}

func main() {
	go OneGoRoutine()
	fmt.Println("main goroutine done!")
}
```

为什么上面的这段代码只输出：main goroutine done!

当你运行这段代码时，可能会遇到两种情况之一：只看到 "main goroutine done!" 的输出，或者同时看到 "Hello Goroutine" 和 "main goroutine done!" 的输出，顺序可能会变。这种行为的不确定性是由于 goroutine 的调度和并发执行的性质导致的。

在你提供的代码中：

```go
func OneGoRoutine() {
	fmt.Println("Hello Goroutine")
}

func main() {
	go OneGoRoutine()
	fmt.Println("main goroutine done!")
}
```

`OneGoRoutine` 函数在一个新的 goroutine 中异步执行。这意味着 `main` 函数不会等待 `OneGoRoutine` 执行完毕，而是立即继续执行下一条语句，打印 "main goroutine done!"。

由于 `main` 函数是程序的入口点，当 `main` 函数执行完毕时，程序将结束，同时所有在执行的 goroutines（无论它们的状态）也会被终止。因此，如果 `main` 函数在 `OneGoRoutine` 函数打印 "Hello Goroutine" 之前就结束了，你将看不到 "Hello Goroutine" 的输出。

要确保看到 "Hello Goroutine" 的输出，你可以在 `main` 函数中添加某种形式的等待（例如，使用 `time.Sleep`，`sync.WaitGroup`，或者更好的方式是使用 channel 来同步），以确保 `main` 函数等待 `OneGoRoutine` 完成：

### 使用 `time.Sleep`：

```go
package main

import (
	"fmt"
	"time"
)

func OneGoRoutine() {
	fmt.Println("Hello Goroutine")
}

func main() {
	go OneGoRoutine()
	time.Sleep(1 * time.Second) // 等待足够长的时间，以确保 goroutine 可以完成
	fmt.Println("main goroutine done!")
}
```

### 使用 `sync.WaitGroup`（更为推荐的方式）：

```go
package main

import (
	"fmt"
	"sync"
)

func OneGoRoutine(wg *sync.WaitGroup) {
	defer wg.Done() // 在函数结束时通知 WaitGroup，当前 goroutine 的工作已完成
	fmt.Println("Hello Goroutine")
}

func main() {
	var wg sync.WaitGroup
	wg.Add(1) // 增加等待 goroutine 的数量
	go OneGoRoutine(&wg)
	wg.Wait() // 等待所有 goroutine 完成
	fmt.Println("main goroutine done!")
}
```

使用 `sync.WaitGroup` 是一种更优雅的同步 goroutines 的方法，它避免了使用 `time.Sleep` 可能导致的不确定性和不必要的延迟。
