defer，顾名思义，就是推迟执行的意思

defer使得后面跟的函数在当前函数结束之后执行（如果有return，就是在reutrn之前执行）

```go
package main

import "fmt"

func func1(a int) int {
    return a + 1
}

func func2(a int) int {
    defer fmt.Println(func1(100))
    fmt.Println("aa")
    return a + 1
}

func main() {
    fmt.Println(func2(10))
}
```

输出

```text
aa
101
11
```

**defer栈：如果有多个defer，则是从后往前执行**（原话：延迟的函数调用被压入一个栈中。当函数返回时， 会按照后进先出的顺序调用被延迟的函数调用。），比如：

```go
package main

import "fmt"

func func1(a int) int {
    return a + 1
}

func func2(a int) int {
    defer fmt.Println(func1(100))
    defer fmt.Println(func1(200))
    fmt.Println("aa")
    return a + 1
}

func main() {
    fmt.Println(func2(10))
}
```

输出

```text
aa
201
101
11
```
