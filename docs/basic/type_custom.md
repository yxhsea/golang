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

2. 普通方式

	```go
	package main

	import "fmt"

	type MyInt int

	func main() {
		var a MyInt
		a = 1
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
		fmt.Println(a, b)
	}
	```

	输出

	```text
	main.MyInt, main.AAInt
	1 2
	```
