# 1 分析下面的函数执行结果以及解释为什么

```go
package main

import (
    "errors"
    "fmt"
)

func foo(a, b int) (i int, err error) {
    defer fmt.Printf("first defer err %v\n", err)
    defer func(err error) { fmt.Printf("second defer err %v\n", err) }(err)
    defer func() { fmt.Printf("third defer err %v\n", err) }()
    if b == 0 {
        err = errors.New("divided by zero!")
        return
    }

    i = a / b
    return
}

func main() {
    foo(2, 0)
}
```

执行结果如下：

```shell
third defer err divided by zero!
second defer err <nil>
first defer err <nil>
```

## 1.1 输出结果

这段代码展示了 `defer` 语句的使用以及如何在 `defer` 语句中捕获函数返回时的变量状态。代码执行的结果和解释如下：

```plaintext
third defer err divided by zero!
second defer err <nil>
first defer err <nil>
```

### 解释

-   **第三个 `defer`**：`defer func() { fmt.Printf("third defer err %v\n", err) }()`。这个 `defer` 语句是一个匿名函数，它捕获了外围函数 `foo` 的局部变量 `err`。由于 `defer` 语句在函数返回之前执行，此时 `err` 已经被赋值为 `errors.New("divided by zero!")`，因此打印出的是更新后的 `err` 值："divided by zero!"。

-   **第二个 `defer`**：`defer func(err error) { fmt.Printf("second defer err %v\n", err) }(err)`。这个 `defer` 语句是通过函数参数显式传递 `err` 变量的当前值（在该 `defer` 语句被注册时的值，即 `nil`）给匿名函数。因为这个时候 `err` 还没有被赋新值（除零错误的赋值发生在之后），所以捕获并打印的 `err` 是 `<nil>`。

-   **第一个 `defer`**：`defer fmt.Printf("first defer err %v\n", err)`。这个 `defer` 语句直接调用 `fmt.Printf`，并且在调用时直接使用了 `err` 的当前值（同样是 `nil`）。因为这个时候 `err` 同样还没有被更新，所以打印出的是 `<nil>`。

### 关键点

-   `defer` 语句在注册时会捕获并保存那一刻的参数值（对于第二个 `defer`）。
-   对于直接在 `defer` 语句中使用的外围函数的变量（如第一个和第三个 `defer`），`defer` 语句执行时会使用这些变量最终的值。
-   因为 `defer` 语句按照后进先出（LIFO）的顺序执行，所以第三个 `defer` 语句首先执行，打印出了更新后的 `err` 值。接着是第二个和第一个 `defer` 语句，它们都打印出了 `err` 的初始值（`<nil>`），因为在它们被注册时 `err` 还没有被更新。

这个例子很好地展示了 `defer` 语句的执行顺序以及它们是如何与外围函数的变量交互的。

## 1.2 为什么会输出这样的结果

理解这个问题的关键在于理解 defer 语句的执行时机和作用域，以及如何捕获变量。

### 第一个 defer

对于 defer fmt.Printf("first defer err %v\n", err)：

在这个 defer 调用中，err 是作为参数直接传递给 fmt.Printf 的。此时，err 的值是在 defer 语句注册时确定的。在 defer 语句注册的时刻，err 还未被赋予新值，因此它的值是 nil。尽管 err 在后续被赋予了新的错误值，但这个变化对于已经注册的 defer 调用来说是不可见的，因为它们捕获的是变量在注册时的值

> 所以这里输出：first defer err <nil>

### 第二个 defer

对于 defer func(err error) { fmt.Printf("second defer err %v\n", err) }(err)：

在这个 defer 调用中，err 同样是在注册 defer 语句时作为参数传递给匿名函数的。这意味着，即使后续 err 变量的值发生了改变，传递给匿名函数的 err 参数（注意这里的参数和外层的 err 变量是两个不同的变量，尽管它们的名字相同）仍然保持在 defer 注册时的值，也就是 nil。

> 所以这里输出：second defer err <nil>

### 第三个 defer

对于 defer func() { fmt.Printf("third defer err %v\n", err) }()：

这个 defer 语句内的匿名函数没有直接接收 err 作为参数，而是闭包形式捕获了外围函数的 err 变量。因此，当匿名函数实际执行时（即 foo 函数返回时），它看到的是 err 当前的值，也就是在 foo 函数内部最后一次对 err 变量赋值后的值

> 所以这里输出：third defer err divided by zero!

### 针对第一个和第二个 defer

为什么 defer fmt.Printf 和 defer func(err error) 打印的都是 nil
简而言之，对于前两个 defer 语句，err 的值是在 defer 语句被注册时确定并捕获的，那时 err 还未被赋新值，所以它们捕获并打印的是 nil。而第三个 defer 使用闭包直接捕获了 err 变量，所以它能够反映 err 最终的值。

## 1.3 所以针对 defer 如果后面的语句直接使用了后面的变量，那么后续这个变量的更新不会被 defer 的语句所感知，对么

实际上，您的理解有一点需要澄清。对于 `defer` 语句，是否能感知到变量后续的更新，取决于变量是如何在 `defer` 语句中被引用的：

1. **直接传递变量作为参数**：如果变量是作为参数直接传递给 `defer` 调用的函数或方法的，则在 `defer` 语句被注册（定义）时就已经对这个变量的当前值进行了评估和捕获。因此，即使后续代码中修改了这个变量的值，这些修改对已经注册的 `defer` 调用是不可见的，它们不会感知到变量的更新。

    ```go
    func example() {
        var err error
        // err 作为参数直接传递给 defer 调用的函数
        defer fmt.Println(err)
        // 修改 err 的值
        err = errors.New("new error")
    }
    // defer 调用时，打印出的 err 仍然是它被注册时的值，即 nil。
    ```

2. **通过闭包引用变量**：如果 `defer` 语句中的函数是通过闭包的形式引用变量的，则这个函数会捕获到变量的引用，而不是变量的值的副本。因此，`defer` 函数执行时，它会看到变量**最后的值**，即函数返回前变量的当前状态。

    ```go
    func example() {
        var err error
        // 通过闭包引用 err 变量
        defer func() {
            fmt.Println(err)
        }()
        // 修改 err 的值
        err = errors.New("new error")
    }
    // defer 调用时，打印出的 err 是它在函数返回前的值，即 "new error"。
    ```

### 总结

因此，如果 `defer` 后面的语句直接使用了变量，并且你希望 `defer` 调用能感知到后续这个变量的更新，你应该确保在 `defer` 语句中通过闭包的方式引用这个变量。这样，无论变量如何更新，`defer` 语句执行时都能反映出最新的变量状态。
