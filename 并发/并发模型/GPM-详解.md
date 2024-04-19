# 1 详细讲解下 GPM

GPM 是 Go 语言运行时（runtime）的核心调度模型，涉及到 Goroutine（G）、Processor（P）和 Machine（M）三个主要部分的交互。理解 GPM 模型有助于深入理解 Go 程序的并发执行机制。

### Goroutine（G）

Goroutine 是 Go 中实现并发的基本单位，比传统的线程更轻量。每个 Goroutine 在执行时只需很小的栈内存（通常几 KB），并且这个栈可以根据需要动态伸缩，这使得创建成千上万个 Goroutine 成为可能。Goroutine 包含要执行的函数信息和其运行时的上下文（如栈指针等）。

### Processor（P）

Processor（P）代表了 Go 运行时对逻辑处理器的抽象，它负责本地化 Goroutine 的执行环境，包含运行 Goroutine 所需的资源。每个 P 都有自己的运行队列，存储待执行的 Goroutine。P 的数量在程序启动时根据 `GOMAXPROCS` 环境变量确定（默认为机器上的 CPU 核心数），限制了同时运行 Goroutine 的数量，从而减少了上下文切换的成本，并允许有效利用多核处理器。

### Machine（M）

Machine（M）代表了操作系统的线程，是实际执行计算的实体。M 在 Go 运行时中扮演着执行器的角色，每个 M 都会绑定一个 P 来执行 P 中队列上的 Goroutine。M 负责 Goroutine 的实际执行，并处理系统调用、调度等任务。当 M 进行系统调用时，P 可以被解绑并分配给其他的 M，这样就不会阻塞其他 Goroutine 的执行。

### GPM 交互

-   **Goroutine 到 Processor 的调度**：当一个 Goroutine 准备好运行时，它会被放入一个 P 的本地运行队列中。如果队列已满，它可能会被移动到全局队列或者其他 P 的本地队列。
-   **Processor 到 Machine 的绑定**：每个 P 通过一个 M 来执行其队列中的 Goroutine。如果一个 M 阻塞（如进行系统调用），运行时会尝试将其对应的 P 转移到另一个 M，以保持 CPU 的利用率。
-   **工作窃取**：为了平衡负载，空闲的 P（通过其 M）会尝试从其他 P 或全局队列中窃取 Goroutine 来执行。

GPM 模型的设计使得 Go 能够高效地利用多核处理器进行并发执行，同时保持调度和系统调用的开销尽可能低。这种模型优化了资源使用，提高了并发性能，是 Go 并发编程强大能力的基石。
