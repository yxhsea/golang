## **函数的传入参数实现接口**

---

```go hl_lines="17 18 19"
package main

import "fmt"

type Reader interface {
	Read() int
}

type MyStruct struct {
	X, Y int
}

func (m *MyStruct) Read() int {
	return m.X + m.Y
}

func run(r Reader) {
	fmt.Println(r.Read())
}

func main() {
	s := &MyStruct{3, 4}
	run(s)
}
```

输出

```text
7
```

可以看出，因为*MyStruct实现了Read方法，因此&MyStruct实现了Reader接口，所以把*MyStruct作为参数调用run时候，&MyStruct{3, 4}就自动实现了Reader接口，因此run里可以用r.Read()来调用接口的方法

## **函数的传出参数实现接口**

---

常用于实例化，如New一个struct

```go hl_lines="17 18 19"
package main

import "fmt"

type Sumer interface {
	Sum() int
}

type MyStruct struct {
	X, Y int
}

func (this *MyStruct) Sum() int {
	return this.X + this.Y
}

func New(a, b int) Sumer {
	return &MyStruct{a, b}
}

func main() {
	m := New(3, 4)
	s := m.Sum()
	fmt.Println(s)
}
```

输出

```text
7
```

可以看出，因为*MyStruct实现了Sum方法，因此&MyStruct实现了Sumer接口

## **空接口也可用于传入和传出参数**

---

```go hl_lines="17 18 19 20 21 22"
package main

import "fmt"

type Reader interface {
	Read() int
}

type MyStruct struct {
	X, Y int
}

func (m *MyStruct) Read() int {
	return m.X + m.Y
}

func run(r interface{}) {
	// 会报错
	//fmt.Println(r.Read())
	fmt.Println(r)
	fmt.Printf("%T\n", r)
}

func main() {
	s := &MyStruct{3, 4}
	run(s)
}
```

输出

```text
&{3 4}
*main.MyStruct
```

如果把上面例子中注释去掉，则会报错

```text
./example.go:19: r.Read undefined (type interface {} is interface with no methods)
```

## **函数传出参数常见的3种形式**

---

```go
func foo(a, b int) SumElementer {
	return &MyStruct{a, b}
}
```

```go
func foo(a, b int) interface{} {
	return &MyStruct{a, b}
}
```

```go
func foo(a, b int) *MyStruct {
	return &MyStruct{a, b}
}
```
