## **reflect.Type**

---

reflect.Type：是一个interface，对应的struct是reflect.\*rtype，在go源码/src/reflect/type.go里定义：

```text
// Type is the representation of a Go type.
```

!!! note
	本文提到的reflect.Type实际均是reflect.\*rtype

!!! warning
	这个reflect.Type是接口hold的变量的类型，而不是追根溯源的最底层类型。比如说

	```text
	type MyStruct struct {}
	```

	这里reflect.Type就是"MyStruct"，而非"struct"。如果想获得struct只能使用Kind()

### **String()**

reflect.Type(实际是reflect.\*rtype)的String()方法：将接口变量hold的变量的类型名转为字符串输出，即输出的变量是类型为string，值为hold的类型名

### **Kind()**

Type和Value都有Kind()方法用来返回一个常量（Type和Value返回的是相同的）

这个常量指示了所hold的值是什么类型 **（这个类型是最终类型，而不是中间类型）**

例如

```text
package main

import (
	"fmt"
	"reflect"
)

type AInt int
type BInt AInt

func main() {
	var e interface{} = BInt(10)
	fmt.Println(reflect.TypeOf(e).Kind())
}
```

输出

```text
int
```


？？？如何获得中间类型？？？

??? note "附Kind()定义"
	下面是Type和Value的Kind()方法中都有提及的核心部分（type Kind）

	```go
	// A Kind represents the specific kind of type that a Type represents.
	// The zero Kind is not a valid kind.
	type Kind uint

	const (
	    Invalid Kind = iota
	    Bool
	    Int
	    Int8
	    Int16
	    Int32
	    Int64
	    Uint
	    Uint8
	    Uint16
	    Uint32
	    Uint64
	    Uintptr
	    Float32
	    Float64
	    Complex64
	    Complex128
	    Array
	    Chan
	    Func
	    Interface
	    Map
	    Ptr
	    Slice
	    String
	    Struct
	    UnsafePointer
	)
	```

## **reflect.Value**

---

reflect.Value：是一个struct，在go源码/src/reflect/value.go里定义：

```text
// Value is the reflection interface to a Go value.
```

### **String()**

reflect.Value的String()方法:

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

**String()例子，包含reflect.Type.String()和reflect.Value.String()**

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

### **Kind()**

[同reflec.Type.Kind()](/reflect/pkg_type_value/#kind)

### **Type()**

reflect.Value有Type()方法，用于得到reflect.Type

```go
package main

import (
	"fmt"
	"reflect"
)

type MyStruct struct {
	X, Y int
}

func main() {
	var e interface{} = MyStruct{3, 4}
	t := reflect.ValueOf(e).Type()
	fmt.Printf("%T %v\n", t, t)
}
```

输出

```text
*reflect.rtype main.MyStruct
```
