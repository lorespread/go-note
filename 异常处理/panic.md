# 1 详解下 panic

在 Go 语言中，`panic` 是一个内置函数，用于处理不可恢复的错误，其作用类似于其他编程语言中的抛出异常。当 `panic` 被调用时，它会立即停止当前函数的执行，开始逐层向上终止，直到达到当前协程的顶层，随后程序会被终止。在这个过程中，`defer` 语句会被正常执行，这为资源清理（如文件关闭、解锁等）提供了机会。

### `panic` 使用场景

-   `panic` 通常用于处理程序中不应该发生的错误，或者在某些错误发生时没有合理的方式来继续执行程序。
-   它也用于处理开发阶段的错误检查，例如检测不应该发生的分支，或者作为未实现功能的占位符。

### 注意事项

-   尽管 `panic` 可以用于异常情况，但推荐的做法是尽可能使用显式的错误返回和处理，而不是依赖于 `panic` 和 `recover`。这样可以保持错误处理的清晰性和可预测性。
-   应谨慎使用 `panic`，因为它导致程序终止运行，不给用户或调用者修复错误的机会。

### 使用示例

下面的例子展示了如何使用 `panic`，以及如何通过 `defer` 和 `recover` 来捕获并处理这种情况，避免程序直接崩溃。

```go
package main

import "fmt"

func mayPanic() {
    panic("a problem occurred")
}

func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered. Error:\n", r)
        }
    }()
    mayPanic()
    fmt.Println("After mayPanic()")
}
```

在这个例子中，`mayPanic` 函数使用 `panic` 抛出了一个错误。在 `main` 函数中，我们使用了 `defer` 关键字来注册一个延迟执行的匿名函数，这个函数内部使用了 `recover`。`recover` 是另一个内置函数，用于捕获并恢复 `panic` 引起的异常状态，防止程序崩溃。如果成功恢复，`recover` 会返回 `panic` 传递的值，否则返回 `nil`。

在这个例子执行时，将会触发 `panic`，然后通过 `defer` 中的 `recover` 函数捕获这个 `panic`，程序将打印出错误信息并正常继续执行。因此，"After mayPanic()" 这行代码不会被执行，因为 `panic` 已经中断了正常的控制流程，但是由于 `recover` 的使用，程序本身不会崩溃。

# 2 为什么 fmt.Println("After mayPanic()") 这行代码不执行

```go
package main

import "fmt"

func mayPanic() {
    panic("a problem occurred")
}

func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered. Error:\n", r)
        }
    }()
    mayPanic()
    fmt.Println("After mayPanic()")
}
```

在你提供的代码示例中，`fmt.Println("After mayPanic()")` 这行代码不执行是因为 `panic` 调用会立即中断当前函数的执行。当 `panic` 被触发时，Go 运行时会停止当前 goroutine 中的正常控制流程，开始逐层向上回溯调用栈，寻找对应的 `defer` 语句执行。这个过程继续进行，直到找到并执行了包含 `recover` 的 `defer` 函数，或者到达了该 goroutine 的顶层并且终止程序执行。

在你的代码中，`mayPanic` 函数中的 `panic` 被调用后，`panic` 开始逐层向上回溯。在这个例子中，它立刻回溯到了 `main` 函数，因为 `mayPanic` 是直接由 `main` 调用的。在回溯过程中，它遇到了 `main` 函数中的 `defer` 函数，该 `defer` 函数包含 `recover` 调用，用于捕获并处理 `panic`。由于 `panic` 导致的控制流程中断，`main` 函数中 `mayPanic` 调用之后的代码（即 `fmt.Println("After mayPanic()")`）不会被执行。

`recover` 函数的执行使得程序可以从 `panic` 状态中恢复，继续执行之后的 `defer` 函数（如果有的话），但是不会回到 `panic` 发生点之后的代码继续执行。因此，即使 `panic` 被成功捕获并处理，`fmt.Println("After mayPanic()")` 这行代码仍然不会执行，因为其执行路径已经被 `panic` 中断了。

总结来说，`panic` 引起的程序控制流程的中断和 `defer` 中的 `recover` 调用能够捕获 `panic`，防止程序立即崩溃，但是不会恢复到 `panic` 发生点之后的代码继续执行。这是为什么 `fmt.Println("After mayPanic()")` 这行代码在你的程序中不会执行的原因。
