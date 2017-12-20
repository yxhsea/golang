## **new()**

---

go内建`new()`的定义

```go
func new(Type) *Type
```

内建函数 new() 用来分配内存，它的第一个参数是一个类型，不是一个值，它的返回值是一个指向新分配类型零值的指针

```go
package main

import "fmt"

type foo struct {
	X, Y int
}

func main() {
	s1 := foo{}
	s2 := new(foo)

	fmt.Printf("s1: %T %v\n", s1, s1)
	fmt.Printf("s2: %T %v\n", s2, s2)
}
```

输出

```text
s1: main.foo {0 0}
s2: *main.foo &{0 0}
```
