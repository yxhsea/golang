枚举就是一一列举，将所有的情况都列举出来，那么取值的时候只能是这几种情况的一种，不能是别的。

go枚举：

在go语言中，没有直接支持枚举的关键字，也就造成go没有直接枚举的功能。但是可以通过const+iota来模拟枚举

例子：

```go
package main

import "fmt"

// 实现枚举例子
type State int

// iota 初始化后会自动递增
const (
    Running State = iota	// value --> 0
    Stopped 				// value --> 1
    Rebooting 				// value --> 2
    Terminated 				// value --> 3
)

func (this State) String() string {
    switch this {
    case Running:
        return "Running"
    case Stopped:
        return "Stopped"
    default:
        return "Unknown"
    }
}

func main() {
    state := Stopped
    fmt.Println("state", state)
}
```

输出

```text
state Stopped
```

没有重载String函数的情况下则输出`state 1`

附iota详解：https://studygolang.com/articles/2192
