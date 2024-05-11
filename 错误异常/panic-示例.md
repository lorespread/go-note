# 1 给出使用 panic 的示例

在 Go 语言中，`error` 和 `panic` 用于不同类型的错误处理场景。理解它们的区别和适用情境是编写健壮和易于维护的 Go 程序的关键。

### 使用 `error`

-   **适用场景**：`error` 用于可以预期并应该由程序逻辑处理的错误。这些错误是正常程序运行的一部分，如文件不存在、无效的用户输入等。
-   **优点**：使用 `error` 可以提供更细粒度的错误处理，允许调用者根据错误信息做出相应的决策。
-   **示例**：

```go
package main

import (
    "errors"
    "fmt"
)

func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

func main() {
    result, err := divide(10, 0)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    fmt.Println("Result:", result)
}
```

### 使用 `panic`

-   **适用场景**：`panic` 用于不可恢复的错误情况，即那些不应该发生或无法通过常规错误处理恢复的错误。例如，程序内部的逻辑错误、不满足的前置条件等。`panic` 通常指示程序中的 bug 或某些致命的问题，需要立即停止执行。
-   **优点**：`panic` 可以立即中断当前函数的执行，逐层向上返回，直到遇到 `recover` 调用。它适用于处理那些程序无法或不应继续执行的严重错误情况。
-   **示例**：

```go
package main

import "fmt"

func mustDoSomething(value int) {
    if value < 0 {
        // 这种情况不应该发生，使用 panic 指示严重错误
        panic("value must be non-negative")
    }
    // 正常的业务逻辑
    fmt.Println("Doing something with value", value)
}

func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
        }
    }()

    mustDoSomething(-1)
    fmt.Println("This line will not run if panic occurs.")
}
```

### 总结

-   使用 `error` 来处理那些预期内的、应该被程序逻辑处理的错误。
-   使用 `panic` 来处理那些不可恢复的、通常是严重的或预期外的错误。`panic` 应该在程序中非常谨慎地使用，因为它涉及到程序的即时终止。
-   通过合理使用 `error` 和 `panic`，可以使你的 Go 程序更加健壮、清晰和易于维护。
