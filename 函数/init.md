# 1 详解下 init 函数

在 Go 语言中，`init` 函数提供了一种特殊的机制，允许在程序执行前对包进行初始化。`init` 函数不需要参数也不返回值，每个包可以有多个 `init` 函数，甚至一个源文件可以包含多个 `init` 函数。`init` 函数的主要用途包括设置包级别的变量、初始化状态、或执行其他只需要运行一次的准备工作。

### 特点

-   **自动执行**：`init` 函数在包被首次导入时自动执行，且在 `main` 函数执行前完成。
-   **执行顺序**：如果一个包导入了其他包，则被导入的包的 `init` 函数先于导入包执行。在同一个包内，`init` 函数按照它们在源文件中出现的顺序执行。
-   **不能被调用或引用**：`init` 函数不能被其他函数调用或直接引用，它们是由 Go 运行时自动调用的。
-   **多个 `init` 函数**：一个包或一个源文件可以包含多个 `init` 函数，它们会按照代码中出现的顺序依次执行。

### 使用示例

假设你正在编写一个库，需要在库被导入并使用之前初始化一些数据结构或者检查必要的环境状态。

**示例 1：初始化包级变量**

```go
package mylib

import "log"

var available bool

func init() {
    // 假设这里有一些检查逻辑来确定库是否可用
    available = true
    log.Println("mylib is initialized and available:", available)
}

// LibraryFunction 是一个示例库函数
func LibraryFunction() {
    if !available {
        log.Println("Library is not available")
        return
    }
    log.Println("Library function is called")
}
```

**示例 2：同一个包中的多个 `init` 函数**

```go
package database

import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql"
)

var db *sql.DB

func init() {
    var err error
    db, err = sql.Open("mysql", "user:password@/dbname")
    if err != nil {
        panic(err)
    }
}

func init() {
    // 执行数据库迁移或检查
    if err := db.Ping(); err != nil {
        panic(err)
    }
}
```

在这个示例中，第一个 `init` 函数用于打开数据库连接，而第二个 `init` 函数用于检查数据库连接是否成功。通过将这些逻辑放在 `init` 函数中，可以保证在包的其他部分使用数据库之前，数据库连接已经建立并且可用。

### 注意事项

-   使用 `init` 函数时应该小心，因为它的执行顺序可能会导致依赖问题或副作用，尤其是在涉及多个包的初始化顺序时。
-   `init` 函数是一种强大的工具，但应谨慎使用。过度依赖 `init` 函数可能会使代码难以理解和维护，特别是当 `init` 函数中包含复杂的逻辑或副作用时。

总的来说，`init` 函数提供了一种便利的方式来进行包级别的初始化，但建议只在确实需要时使用，并保持 `init` 函数的简洁和明确。
