## **reflect.Type**

---

reflect.Type：是一个interface，在go源码/src/reflect/type.go里定义：

```text
// Type is the representation of a Go type.
```

## **reflect.Value**

---

reflect.Value：是一个struct，在go源码/src/reflect/value.go里定义：

```text
// Value is the reflection interface to a Go value.
```

## **String()**

---

- reflect.Type(实际是reflect.\*rtype)的String()方法：将接口变量hold的变量的类型名转为字符串输出，即输出的变量是类型为string，值为hold的类型名

- reflect.Value的String()方法:

	- 若hold的变量类型是string则输出值

	- 若hold的变量类型不是string则输出`<接口hold的变量的类型名 Value>`

		??? note "例子"
			```text
			var x float64 = 3.4
			fmt.Println("value:", reflect.ValueOf(x).String())
			```

			输出

			```text
			value: <float64 Value>
			```

		??? note "附String()定义"
			```go
			// String returns the string v's underlying value, as a string.
			// String is a special case because of Go's String method convention.
			// Unlike the other getters, it does not panic if v's Kind is not String.
			// Instead, it returns a string of the form "<T value>" where T is v's type.
			// The fmt package treats Values specially. It does not call their String
			// method implicitly but instead prints the concrete values they hold.
			func (v Value) String() string {
			    switch k := v.kind(); k {
			    case Invalid:
			        return "<invalid Value>"
			    case String:
			        return *(*string)(v.ptr)
			    }
			    // If you call String on a reflect.Value of other type, it's better to
			    // print something than to panic. Useful in debugging.
			    return "<" + v.Type().String() + " Value>"
			}
			```

		为什么要输出`<接口hold的变量的类型名 Value>`，是因为在go里非string类型无法转化为string，若直接panic又不太友好，同时为了便于调试，干脆就打印类型好了（即调用`.Type()`）

例子

```go
package main

import (
	"fmt"
	"reflect"
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
	var i Adder = MyStruct{3, 4}
	t1 := reflect.TypeOf(i)
	v1 := reflect.ValueOf(i)
	st1 := t1.String()
	sv1 := v1.String()

	fmt.Printf("i:\t%T\t%v\n", i, i)
	fmt.Printf("t1:\t%T\t%v\n", t1, t1)
	fmt.Printf("v1:\t%T\t%v\n", v1, v1)
	fmt.Printf("st1:\t%T\t\t%v\n", st1, st1)
	fmt.Printf("sv1:\t%T\t\t%v\n", sv1, sv1)

	fmt.Println("-----")

	var e interface{} = "hello, world"
	v2 := reflect.ValueOf(e)
	sv2 := v2.String()
	fmt.Printf("e:\t%T\t\t%v\n", e, e)
	fmt.Printf("v2:\t%T\t%v\n", v2, v2)
	fmt.Printf("sv2:\t%T\t\t%v\n", sv2, sv2)
}
```

输出

```text
i:		main.MyStruct	{3 4}
t1:		*reflect.rtype	main.MyStruct
v1:		reflect.Value	{3 4}
st1:	string			main.MyStruct
sv1:	string			<main.MyStruct Value>
-----
e:		string			hello, world
v2:		reflect.Value	hello, world
sv2:	string			hello, world
```
