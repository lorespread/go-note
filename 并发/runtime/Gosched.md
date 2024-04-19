# 1 详解下 runtime 的 Gosched

`runtime.Gosched()` 是 Go 语言 `runtime` 包提供的一个函数，它的作用是让当前 goroutine 让出 CPU 时间片给其他 goroutine 运行，但不会挂起当前 goroutine。当前 goroutine 之后会在未来的某个时间点继续执行。这个函数是协作式调度的一部分，允许开发者在长时间运行的操作中手动地放弃 CPU 使用权，以避免 monopolizing CPU 资源。

### 使用场景

`runtime.Gosched()` 主要在以下场景中使用：

-   **性能优化**：在计算密集型的操作中，如果没有适当的等待或阻塞（比如 I/O 操作），可能会导致单个 goroutine 占用过多 CPU 时间，从而影响到其他 goroutine 的执行。通过调用 `runtime.Gosched()`，可以使 CPU 更公平地分配给其他 goroutine。
-   **避免死锁**：在某些复杂的同步场景下，手动调用 `runtime.Gosched()` 可以帮助避免死锁，尤其是在循环等待资源时。

### 使用示例

下面是一个使用 `runtime.Gosched()` 的简单示例：

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func task(id int) {
    for i := 0; i < 3; i++ {
        fmt.Printf("Task %d: iteration %d\n", id, i)
        // 让出 CPU 时间片，给其他 goroutine 运行的机会
        runtime.Gosched()
    }
}

func main() {
    go task(1)
    go task(2)

    // 等待足够的时间，确保上面的 goroutine 执行完成
    time.Sleep(time.Second)
}
```

在这个例子中，两个 goroutine 交替执行，每次迭代后通过 `runtime.Gosched()` 主动让出 CPU 时间片。这确保了两个 goroutine 能够在它们各自完成前，相对公平地共享 CPU 资源。

### 注意事项

-   调用 `runtime.Gosched()` 并不保证其他 goroutine 将立即执行，它只是提供了调度的机会。
-   过多地使用 `runtime.Gosched()` 可能会导致程序性能下降，因为频繁地切换上下文也是有开销的。
-   在大多数情况下，合理设计 goroutine 和 channel 的同步机制，可以避免手动使用 `runtime.Gosched()`，让 Go 的调度器自动优化执行效率。

通常，Go 的运行时调度器能够很好地管理 goroutine 的执行，无需手动干预。但在特定场景下，了解并合理使用 `runtime.Gosched()` 可以帮助开发者更细致地控制程序的并发行为。

# 2 详解 2

`runtime.Gosched()` 是 Go 语言的运行时系统提供的一个函数，它是 `runtime` 包的一部分。这个函数的作用是让当前 goroutine 让出 CPU 时间给其他 goroutine 运行，但并不会挂起当前 goroutine，当前 goroutine 在未来的某个时间点将继续执行。

### 功能和工作原理

在 Go 的并发模型中，goroutines 是由 Go 运行时管理的轻量级线程。它们在一组逻辑处理器上运行，由运行时的调度器进行调度。默认情况下，Go 运行时为每个可用的物理处理器分配一个逻辑处理器（P）。每个逻辑处理器上运行一个内核线程（M），并在其上调度多个 goroutines。

当一个 goroutine 执行 `runtime.Gosched()` 时，它会告诉调度器放弃当前的时间片，暂时将 CPU 让给其他等待运行的 goroutines。此时调度器会选择另一个 goroutine 继续执行，而原 goroutine 会放回到本地 run queue（如果有 P 绑定的话）或全局 run queue，等待下一次调度。

### 使用场景

`runtime.Gosched()` 主要在以下几种场景中使用：

1. **防止长时间占用 CPU 导致其他 goroutine 饿死**：在一些需要长时间运行的循环中，如果没有足够的 I/O 操作或其他阻塞调用来自然地让出 CPU，使用 `runtime.Gosched()` 可以强制让出 CPU 时间，确保其他 goroutine 也有机会运行。

2. **优化性能**：在高度并发的应用中，合理的使用 `runtime.Gosched()` 可以改善程序的响应性和性能，尤其是在单个 goroutine 可能会因为某些计算密集型任务而长时间占用 CPU 时。

3. **提升调度公平性**：在某些特定的计算或循环中，手动插入 `runtime.Gosched()` 可以帮助调度器更公平地分配时间片给其他等待执行的 goroutines。

### 示例代码

以下是一个使用 `runtime.Gosched()` 的示例：

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
)

func main() {
	var wg sync.WaitGroup
	wg.Add(2)

	// Goroutine 1
	go func() {
		defer wg.Done()
		for i := 0; i < 3; i++ {
			fmt.Println("Goroutine 1")
			runtime.Gosched() // 让出 CPU，允许其他 goroutine 运行
		}
	}()

	// Goroutine 2
	go func() {
		defer wg.Done()
		for i := 0; i < 3; i++ {
			fmt.Println("Goroutine 2")
			runtime.Gosched() // 让出 CPU，允许其他 goroutine 运行
		}
	}()

	wg.Wait()
}
```

在这个示例中，两个 goroutine 交替执行，每次打印后都调用 `runtime.Gosched()` 来主动让出 CPU。这使得两个 goroutine 能够在输出上看起来像是在并行运行，尽管它们是在单核处理器上通过时间片轮转调度的。

### 总结

`runtime.Gosched()` 是一个有用的工具，用于控制 goroutine 的调度。虽然大多数情况下 Go 的调度器可以自动处理 goroutine 之间的调度，但在特定情况下，手动调用 `runtime.Gosched()` 可以提高程序的并发性能和响应速度。

# 3 what's the difference between runtime.Gosched and time.Sleep

`runtime.Gosched()` 和 `time.Sleep()` 都是在 Go 中用来影响 goroutine 行为的函数，但它们的用途和效果有着明显的区别：

### 1. **目的和行为**

-   **runtime.Gosched()**

    -   **用途**: `runtime.Gosched()` 用于让出当前 goroutine 的 CPU 时间片，允许其他 goroutine 运行，但并不会挂起当前 goroutine。它仅是将当前 goroutine 重新放入调度队列中，等待下一次调度。
    -   **行为**: 当一个 goroutine 调用 `runtime.Gosched()` 后，它会立即停止执行当前的代码，调度器会从队列中选择另一个 goroutine 继续执行。当前 goroutine 会在未来某个时刻继续执行，具体取决于调度器的决定和系统的负载。

-   **time.Sleep()**
    -   **用途**: `time.Sleep(duration)` 用于使当前 goroutine 在指定的时间内暂停执行。它实际上是在阻塞当前 goroutine，防止它在指定的时间内执行任何操作。
    -   **行为**: 调用 `time.Sleep()` 会挂起当前 goroutine，直到经过指定的时间后，它才可能被调度器重新唤醒。在睡眠期间，当前 goroutine 完全不会执行任何代码。

### 2. **调度影响**

-   **runtime.Gosched()** 通过主动放弃 CPU 时间片，有助于改善应用的响应性和多任务处理能力，尤其是在处理多个运行密集型任务时。它不会导致 goroutine 阻塞，只是允许调度器重新调度。
-   **time.Sleep()** 实际上引入了阻塞，使得 goroutine 在一段时间内不参与 CPU 时间的竞争。这对于延迟、定时任务、等待外部事件无法快速响应的场景较为适用。

### 3. **使用场景**

-   使用 `runtime.Gosched()` 通常见于需要释放 CPU 以避免长时间占用 CPU 资源，从而允许其他 goroutine 运行的场景，特别是在循环或计算密集型操作中。
-   `time.Sleep()` 通常用于实现延迟、简单的定时功能，或者在等待外部资源时简单地阻塞 goroutine。

### 示例

假设你有一个需要频繁更新的状态，但希望确保其他 goroutine 也有执行机会：

```go
// 使用 runtime.Gosched()
for {
    updateStatus()
    runtime.Gosched() // 允许其他 goroutine 运行
}

// 使用 time.Sleep()
for {
    updateStatus()
    time.Sleep(100 * time.Millisecond) // 每次更新后暂停一段时间
}
```

在第一个示例中，使用 `runtime.Gosched()` 是为了确保在频繁更新过程中，其他 goroutine 也可以得到执行的机会。而第二个示例中使用 `time.Sleep()` 是为了减慢更新频率，适用于不需要即时响应的情况。

### 总结

`runtime.Gosched()` 和 `time.Sleep()` 虽然都可以影响 goroutine 的执行，但适用的场景不同。`runtime.Gosched()` 更多的是为了调度的公平性和响应性，而 `time.Sleep()` 是为了实现延迟和定时控制。根据应用的具体需求选择合适的方法。

# 4 下面的代码执行打印后为什么没有打印 child goroutine run

示例代码：

```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	go func() {
		fmt.Println("child goroutine run")
	}()

	runtime.Gosched()
	fmt.Println("main goroutine run")
}
```

执行该代码段，几乎没有机会看到子协程的输出。这是因为目前大部分的计算机均为多核 CPU，默认情况下，Go 语言也会申请多个内核线程进行处理，主协程和子协程大概率是不会在同一个 CPU 核心上执行的。所以，即使主协程让出了时间片，也不会对子协程的执行有任何影响，反而主协程会很快被重新调度执行。在子协程中的 println() 语句来不及输出的情况下，主协程便已结束，从而导致整个应用程序的退出。

🔥：也就是说当使用 runtime.Gosched，主线程让出 CPU 时，发现没有其他协程需要执行，就会理解切换回执行主线程，在多核的情况下，主线程可能是在 A 核上跑，其他协程可能是在其他 B 核上跑，这就可能会出现主线程已经执行完毕，但是其他协程还没有执行完毕，所以就看不到子协程的打印，对否

您的理解是正确的。在 Go 语言的并发模型中，`runtime.Gosched()` 的作用确实是建议调度器让出当前 goroutine 正在使用的 CPU 时间片，以便其他等待运行的 goroutines 可以得到执行的机会。但是，这个“让出”操作的效果和确切行为取决于多个因素，特别是在多核处理器环境中。

### 运行环境的影响

1. **多核处理器**：在多核处理器上，Go 的调度器会尝试在可用的逻辑处理器（P）上分配运行 goroutines 的内核线程（M）。如果主 goroutine 和其他 goroutines 被分配到不同的核上执行，它们实际上是并行运行的。当主 goroutine 调用 `runtime.Gosched()` 时，它确实会暂时让出其正在使用的处理器的时间片，但由于其他核上的 goroutine 可能还未开始或正在执行，调度器可能很快就会再次调度主 goroutine 继续执行，尤其是如果没有足够的其他工作负载时。

2. **调度器行为**：Go 的调度器采用的是工作窃取算法，当一个处理器上没有足够的 goroutine 运行时，它会尝试从其他处理器“窃取”一些 goroutines 来执行。这种机制意味着在调用 `runtime.Gosched()` 后，调度器的确会尝试重新分配执行时间给其他 goroutines，但这不保证立即的上下文切换，尤其是在当前处理器的本地队列中如果没有其他就绪的 goroutine。

### 实际表现

在实践中，如果主 goroutine 在调用 `runtime.Gosched()` 后紧接着执行结束，并且主函数返回，则整个程序会终止，包括所有尚未执行完毕的 goroutines。这就可能导致你观察到的现象：主 goroutine 完成执行，程序退出，而其他 goroutine 还没有机会运行或完成它们的打印任务。

### 更好的同步机制

为了确保在主 goroutine 中可以观察到其他 goroutines 的执行结果，你应该使用像 `sync.WaitGroup` 这样的同步机制来控制主 goroutine 的退出时机，保证所有子 goroutines 都有机会执行完毕：

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
)

func main() {
	var wg sync.WaitGroup
	wg.Add(1)

	go func() {
		defer wg.Done()
		fmt.Println("child goroutine run")
	}()

	runtime.Gosched()
	wg.Wait() // 等待所有 goroutine 完成
	fmt.Println("main goroutine run")
}
```

这样，你可以确保子 goroutine 的输出在主 goroutine 结束前被执行和观察到，从而避免了程序过早退出导致的问题。

### 如何修改

我们可以调用 runtime.GOMAXPROCS(1)，将最大 Processor 的数量限制为 1，并再次进行测试，如代码清单 9-10 所示。代码清单 9-10 限制 Processor 数量为 1 时，主协程利用 runtime.Gosched() 让出时间片

```go
func main() {
	runtime.GOMAXPROCS(1)
	go func() {
		fmt.Println("child goroutine run")
	}()

	runtime.Gosched()
	fmt.Println("main goroutine run")
}
```

经过测试，会发现此时子协程成功执行的概率大大增加。在笔者的计算机上，大概有 50% 的概率能够打印出字符串“子协程执行”。在主协程主动让出 CPU 的情况下，子协程也未必能够执行完成，这表示子协程来不及打印字符串便又经历了一次 CPU 时间片轮转。

修改后的执行结果如下：

```shell
> go run runtime/gosched1.go
child goroutine run
main goroutine run
```
