# 1 如何自定义 error，给出对应的示例

在 Go 语言中，自定义错误可以通过实现 `error` 接口来创建，这需要定义一个类型并为其实现 `Error() string` 方法。自定义错误使你能够提供更多的上下文信息和错误处理的灵活性。以下是如何自定义错误的步骤及示例：

### 定义自定义错误类型

你可以定义一个结构体来包含错误的详细信息，然后为该结构体实现 `Error() string` 方法。

```go
package main

import "fmt"

// MyError 是一个自定义的错误类型，它实现了 error 接口。
type MyError struct {
    Message string // 错误描述
    Code    int    // 错误代码
}

// Error 方法使 MyError 类型满足 error 接口。
// 这里返回了错误的描述信息。
func (e *MyError) Error() string {
    return fmt.Sprintf("Error %d: %s", e.Code, e.Message)
}

// 触发一个自定义错误的函数。
func doSomething() error {
    // 这里根据某些逻辑判断返回自定义的错误。
    // 例如，这里假设总是返回一个错误。
    return &MyError{
        Message: "Something went wrong",
        Code:    404,
    }
}

func main() {
    err := doSomething()
    if err != nil {
        fmt.Println(err)
    }
}
```

### 使用自定义错误

在上面的示例中，`doSomething` 函数返回了一个 `*MyError` 类型的错误。在 `main` 函数中，通过判断 `err != nil` 来检查是否发生了错误，如果有错误发生，则打印出错误信息。因为 `MyError` 实现了 `error` 接口，所以它可以像任何其他错误一样被处理。

### 利用自定义错误提供更多上下文

自定义错误的一个主要优点是能够提供比普通错误更丰富的上下文信息。比如在上面的 `MyError` 类型中，除了错误信息外，还包含了一个错误代码，这可以用来表示错误的类型或原因，让错误处理更加灵活和详细。

### 检查自定义错误类型

有时你可能需要根据错误的具体类型来做出不同的处理决策。你可以使用类型断言或 `errors.As` 函数来检查错误是否为特定的自定义类型：

```go
if myErr, ok := err.(*MyError); ok {
    fmt.Println("Custom error code:", myErr.Code)
}
```

或者使用 `errors.As`：

```go
var myErr *MyError
if errors.As(err, &myErr) {
    fmt.Println("Custom error code:", myErr.Code)
}
```

通过自定义错误和上述技巧，你可以构建出更加健壮和易于维护的错误处理逻辑。
