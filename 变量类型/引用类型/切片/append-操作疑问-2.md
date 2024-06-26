```go
package main

import "fmt"

func main() {

	fmt.Println("------------------测试 append 追加，底层数组地址-------------------")
	s := make([]int, 5, 10)
	fmt.Printf("切片新建状态，底层数组地址：%p\n", s)

	s2 := append(s, 5, 6, 7)
	// fmt.Printf("切片追加状态，未超过当前容量，底层数组地址：%p\n", _)

	s = append(s, 10, 11)
	fmt.Printf("切片追加状态，超过当前容量，底层数组地址：%p\n", s)
	fmt.Println(s)
	fmt.Println(s2)

}
```

为什么输出的结果如下

```shell
切片新建状态，底层数组地址：0xc000024320
切片追加状态，超过当前容量，底层数组地址：0xc000024320
[0 0 0 0 0 10 11]
[0 0 0 0 0 10 11 7]
```

在你提供的代码中，`s` 和 `s2` 打印出相同的结果可能是一个误解，实际上它们应该显示不同的内容。让我们逐步分析代码和行为来确保理解准确。

### 代码分析

你的代码如下：

```go
s := make([]int, 5, 10)
fmt.Printf("切片新建状态，底层数组地址：%p\n", s)

s2 := append(s, 5, 6, 7)
// 这里 s2 应该是 [0, 0, 0, 0, 0, 5, 6, 7]

s = append(s, 10, 11)
// 这里 s 应该是 [0, 0, 0, 0, 0, 10, 11]
fmt.Printf("切片追加状态，超过当前容量，底层数组地址：%p\n", s)
fmt.Println(s)
fmt.Println(s2)
```

### 预期的结果

-   **s** 初始化后是 `[0, 0, 0, 0, 0]`，容量为 10。
-   **s2** 是将 `5, 6, 7` 追加到 `s` 后得到的新切片，理论上它的内容应该是 `[0, 0, 0, 0, 0, 5, 6, 7]`。
-   然后，再次对 `s` 追加 `10, 11`，因为原 `s` 容量充足，此操作不应导致新的内存分配，所以 `s` 的内容应该是 `[0, 0, 0, 0, 0, 10, 11]`。

### 关于底层数组

-   当 `s2 := append(s, 5, 6, 7)` 执行时，`s` 的底层数组被扩展，但这仅增加了元素 `5, 6, 7`。
-   当 `s = append(s, 10, 11)` 执行时，这种操作通常应该在原有的数组上追加 `10, 11`，替换原本由 `s2` 添加的 `5, 6`。但由于 `s` 重新赋值没有超出容量，所以它不会创建新的底层数组。

### 为什么 `s` 和 `s2` 有部分数据看起来相同

如果 `s` 和 `s2` 打印的结果有部分数据相同，并且这个相同的结果是 `[0, 0, 0, 0, 0, 10, 11]`，这可能是因为：

1. **后续的修改影响了前一个切片**：由于 `s` 和 `s2` 共享相同的底层数组，后对 `s` 的修改（在容量允许的情况下追加的 `10, 11`）实际上覆盖了 `s2` 中的 `5, 6`。

2. **切片操作的时序和打印输出**：在 Go 中，尤其是在某些环境下（如某些在线运行器或者特定的并发环境），输出的时序可能导致看起来两个输出相同。

### 实验验证

你可以通过在每个步骤打印切片内容和底层数组的实际地址来验证这一点：

```go
fmt.Println(s, s2)
fmt.Printf("底层数组地址：%p, %p\n", &s[0], &s2[0])
```

这将帮助确认是否真的发生了底层数组的共享和覆盖。实际上，在正常的操作下，按照上述的逻辑和 Go 的行为，`s` 和 `s2` 应该显示不同的结果，除非有非预期的操作或环境因素干扰。
