## **声明**

---

```text
type 自定义类型 原始类型
```

例如

```text
type MyInt int
```

!!! note ""
	声明可以放在包级别也可以放在函数级别

原始类型也可以是自定义类型，例如

```text
type MyInt int
type AbInt MyInt
```

可以缩写为

```text
type (
	MyInt int
	AbInt MyInt
)
```

## **赋值**

---

有2种方式:

1. 使用强制类型转换的方法来赋值

	```text
	类型(值)
	```

	例如

	```go
	package main

	import "fmt"

	type MyInt int

	func main() {
		a := MyInt(1)
		fmt.Printf("%T, %d\n", a, a)
	}
	```

	输出

	```text
	main.MyInt, 1
	```

2. literal方式

	```go
	package main

	import "fmt"

	type MyInt int

	func main() {
		var a MyInt = 1
		fmt.Printf("%T, %d\n", a, a)
	}
	```

	输出

	```text
	main.MyInt, 1
	```

## **注意**

---

1. 原始类型可以是另一个自定义类型，例如

	```go
	package main

	import "fmt"

	type (
		MyInt int
		AbInt MyInt
	)

	func main() {
		a := MyInt(1)
		b := AbInt(2)
		fmt.Printf("%T, %T\n", a, b)
	}
	```

	输出

	```text
	main.MyInt, main.AbInt
	```

2. 自定义类型不能与原始类型进行计算

	```go
	package main

	import "fmt"

	type MyInt int

	func main() {
		a := MyInt(1)
		b := 2
		fmt.Println(a + b)
	}
	```

	报错

	```text
	./type_custom.go:10: invalid operation: a + b (mismatched types MyInt and int)
	```

	```go
	package main

	import "fmt"

	type (
		MyInt int
		AAInt MyInt
	)

	func main() {
		a := MyInt(1)
		b := AAInt(2)
		fmt.Printf("%T, %T\n", a, b)
		fmt.Println(a, b)
		fmt.Println(a+b)
	}
	```

	报错

	```text
	./type_custom.go:15: invalid operation: a + b (mismatched types MyInt and AAInt)
	```

3. `fmt.Printf`的`%d`可以用于最终原始类型是int的MyInt

	```go
	package main

	import "fmt"

	type (
		MyInt int
		AAInt MyInt
	)

	func main() {
		a := MyInt(1)
		b := AAInt(2)
		fmt.Printf("%T, %T\n", a, b)
		fmt.Printf("%d, %d\n", a, b)
	}
	```

	输出

	```text
	main.MyInt, main.AAInt
	1 2
	```

4. 可以把type写在main里，比如：

	```go
	func main() {
	    type cc int
	    type dd cc
	    var c cc
	    var d dd
	    fmt.Printf("%T\n", c)
	    fmt.Printf("%T\n", d)
	    fmt.Println(c)
	    fmt.Println(d)
	}
	```

	输出

	```text
	main.cc
	main.dd
	0
	0
	```

	但是只有放在包级别的，并且首字母大写的，才能被导入给其他模块引用

5. 自定义type具有隐藏原type的效果

	```go
	type foo int
	type bar []int

	func main() {
	    a := []foo{1, 2, 3}
	    fmt.Printf("%T\n", a)
	    fmt.Println(a)

	    b := bar{1, 2, 3}
	    fmt.Printf("%T\n", b)
	    fmt.Println(b)
	}
	```

	输出

	```text
	[]main.foo
	[1 2 3]
	main.bar
	[1 2 3]
	```

	foo和bar最终是相似效果，但bar通过%T看不到[]

6. TYPE(同类型的值)可以做类型转换，比如：

	```go
	type foo int
	a := foo(1)

	type bar []int
	c := []int{1, 2, 3}
	d := bar(c)
	```

	上面的类型转换都是将原来的值转换为新的类型，只不过新的类型和原来的类型其实是一样的而已。

	**当然也能将不同类型的做转换（前提是golang允许转换的），比如string(1)，就是将int类型的1转换成string类型，只不过不是转换成"1"，而是其他算法（ascii或者utf8？），但反过来int("1")就会报错**

	也可以将int转成float64，或者将floaf64转成int类型
