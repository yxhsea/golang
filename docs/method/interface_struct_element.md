struct里的元素可以是接口类型，举例如下：

```go
package main

import "fmt"

type Reader interface {
	Read() int
}

type MyStruct struct {
	X Reader
	Y int
}

type Foo struct {
	a, b int
}

func (f *Foo) Read() int {
	return f.a + f.b
}

func main() {
	ms := &MyStruct{}
	fmt.Printf("%T %v\n", ms, ms)

	ms.Y = 3
	fmt.Printf("%T %v\n", ms, ms)

	ms.X = &Foo{8, 9}
	fmt.Printf("%T %v\n", ms, ms)
	fmt.Printf("%T %v\n", ms.X, ms.X)
	fmt.Printf("%T %v\n", ms.X.Read(), ms.X.Read())
}
```

输出

```text
*main.MyStruct &{<nil> 0}
*main.MyStruct &{<nil> 3}
*main.MyStruct &{0xc42000e290 3}
*main.Foo &{8 9}
int 17
```
