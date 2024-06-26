在 Go 语言中，`rune` 不是一个缩写，而是一个专门的术语。`rune` 类型在 Go 中用于表示一个 Unicode 码点（Code Point）。这个名字来源于古英语和北欧语言，其中 "rune" 字面意思是“秘密”或“神秘的符号”，用来指代古代北欧的字母系统中的字符。

在计算机编程的上下文中，Go 语言选择使用 `rune` 这个词来表示 Unicode 字符，强调每个 `rune` 值代表一个 Unicode 码点。`rune` 类型实际上是 `int32` 的别名，能够表示 Unicode 码点的全范围，从 U+0000 到 U+10FFFF。

因此，`rune` 在 Go 中专门用于处理 Unicode 文本，使得对国际化文本的处理变得更加方便和直接。
