函数值(Function values): 函数也是值。他们可以像其他值一样传递，比如，函数值可以作为函数的参数或者返回值。

在其他语言里，函数值叫做函数指针(Function pointer)，常用于回调和闭包

举例

```go
package main

import "fmt"

func bb(x, y int) int {
    return x + y
}

func main() {
    cc := func(x, y int) int {
        return x + y
    }
    fmt.Printf("%T\n", bb)
    fmt.Printf("%T\n", cc)
    fmt.Println(bb(1, 2))
    fmt.Println(cc(1, 2))
}
```

输出

```text
func(int, int) int
func(int, int) int
3
3
```

结论:

1. 函数bb的类型和变量cc的类型相同，都是func(int, int) int

2. 说明bb和cc是相同的，只是写法不同，但实质是相同的，而且使用起来效果也一样
