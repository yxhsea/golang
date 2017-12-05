## **基础**

---

type assertions类型断言：

限制：**只能用于interface的值**

```text
t := i.(T)
```

断言interface i的值类型是T，如果是，t则为interface i的值，并且t的类型也为interface i的值类型；如果不是，则panic报错，类似：panic: interface conversion: interface {} is int, not float64

```text
t, ok := i.(T)
```

断言interface i的值类型是T，如果是，t则为interface i的值，并且t的类型也为interface i的值类型，ok为布尔类型的true；如果不是，不会报panic，而是ok为false，同时t的值类型为T，t的值为类型T的初始值（比如int、float64的初始值为0，string初始值为空字符串）

例子：

```go
package main

import "fmt"

func main() {
	var i interface{} = 100

	s, ok := i.(int)
	fmt.Println(s, ok)
	fmt.Printf("%T\n", s)

	f, ok := i.(float64)
	fmt.Println(f, ok)
	fmt.Printf("%T\n", f)
}
```

输出

```text
100 true
int
0 false
float64
```

**类型断言是个好东西，可以将空接口变量hold的变量重新赋值给非空接口**

```go
package main

import (
	"fmt"
)

type Adder interface {
	Add()
}

type MyStruct struct {
	X, Y int
}

func (this MyStruct) Add() {
	fmt.Println(this.X + this.Y)
}

func main() {
	s := MyStruct{3, 4}
	var e interface{}
	e = s
	fmt.Printf("e: %T %v\n", e, e)
	fmt.Println("-----")

	// 写法1: 类型断言写法
	x, ok := e.(MyStruct)
	if ok == true {
		fmt.Printf("x: %T %v\n", x, x)
		x.Add()
	}
	var i Adder
	i = x
	fmt.Printf("i: %T %v\n", i, i)
	i.Add()
	fmt.Println("-----")

	// 写法2: 类型断言+实现接口写法，支持空接口。该写法是写法1的简写方式
	var j Adder
	j = e.(Adder)
	fmt.Printf("j: %T %v\n", j, j)
	j.Add()
	fmt.Println("-----")

	// 写法3: 类型断言+实现接口写法，支持非空接口
	var k Adder
	k = i.(Adder)
	fmt.Printf("k: %T %v\n", k, k)
	k.Add()
}
```

输出

```text
e: main.MyStruct {3 4}
-----
x: main.MyStruct {3 4}
7
i: main.MyStruct {3 4}
7
-----
j: main.MyStruct {3 4}
7
-----
k: main.MyStruct {3 4}
7
```

## **类型断言实现接口**

---

[详见各种实现章节](/method/interface_implement_interface/#_5)

## **类型断言获得承接的指针变量指向的值**

---

[详见空接口承接指针变量](/method/interface_empty/#_2)
