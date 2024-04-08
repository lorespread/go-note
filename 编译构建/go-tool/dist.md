`go tool dist list` 是 Go 语言命令行工具中的一个命令，用于列出 Go 语言支持的所有操作系统（GOOS）和架构（GOARCH）的组合。这个命令非常有用，尤其是在进行交叉编译时，你需要知道目标平台的准确 GOOS 和 GOARCH 值。

### 查看支持的系统和架构

当你运行 `go tool dist list` 命令时，它会输出 Go 语言支持的所有平台，格式为 `GOOS/GOARCH` 的组合，每个组合占一行

```go
> go tool dist list
aix/ppc64
android/386
android/amd64
android/arm
android/arm64
darwin/amd64
darwin/arm64
dragonfly/amd64
freebsd/386
freebsd/amd64
freebsd/arm
freebsd/arm64
freebsd/riscv64
illumos/amd64
ios/amd64
ios/arm64
js/wasm
linux/386
linux/amd64
linux/arm
linux/arm64
linux/loong64
linux/mips
linux/mips64
linux/mips64le
linux/mipsle
linux/ppc64
linux/ppc64le
linux/riscv64
linux/s390x
netbsd/386
netbsd/amd64
netbsd/arm
netbsd/arm64
openbsd/386
openbsd/amd64
openbsd/arm
openbsd/arm64
openbsd/ppc64
plan9/386
plan9/amd64
plan9/arm
solaris/amd64
wasip1/wasm
windows/386
windows/amd64
windows/arm
windows/arm64
```

### 使用 `go tool dist list`

```sh
darwin/amd64
linux/arm
windows/386
```

这个列表表示 Go 语言可以为 macOS（darwin）的 AMD64 架构、Linux 的 ARM 架构、Windows 的 386 架构等平台构建程序。

### 应用场景

-   **交叉编译**：当你想为不同的平台构建你的 Go 程序时，`go tool dist list` 可以帮助你找到正确的 GOOS 和 GOARCH 组合。然后，你可以在 `go build` 命令中设置这些值来生成相应平台的可执行文件。

-   **了解支持情况**：对于想了解 Go 语言对新平台的支持情况，或者检查某个特定平台是否被支持的开发者和用户来说，这个命令提供了一个快速查看的方式。

### 示例：为 ARM 架构的 Linux 构建程序

假设你有一个 Go 程序，并想为 ARM 架构的 Linux 设备构建它。首先，使用 `go tool dist list` 来找到对应的 GOOS/GOARCH 组合：

```sh
go tool dist list | grep linux/arm
```

假设输出显示 `linux/arm` 是支持的组合之一，然后你可以使用以下命令进行交叉编译：

```sh
GOOS=linux GOARCH=arm go build -o your_program_linux_arm your_program.go
```

这会生成一个为 ARM 架构的 Linux 设备准备的可执行文件 `your_program_linux_arm`。

### 结论

`go tool dist list` 是一个实用的命令，对于涉及多平台支持和交叉编译的开发工作非常有帮助。它提供了一个清晰的视图，让你可以轻松地了解和选择 Go 语言支持的平台目标。
