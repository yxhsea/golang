## **基本格式**

---

```go
if x > 0 {
    ...
}
```

`支持`
```go
if x > 0 {
	...
} else if x < 0 {
	...
} else {
	...
}
```

## **短声明**

---

```go
if v := math.Pow(x, n); v < lim {
    ...
}
```

!!! warning ""
	v是局部变量，只在if语句里有效（包括else）

并不是必须要用短声明`:=`，比如下面这样也是可以的

```go
var a int
if a = 1; a < 2 {
    fmt.Println("yes")
}
```

!!! error "注意"
	无论a < 2是否为true，a = 1都被执行了。因此谨慎在if条件句里引用外部变量做动作

`else if`也可以带短声明，但`else`不能带短声明

```go
if x := 0; a > 0 {
	...
} else if x := 1; a < 0 {
	...
} else {
	...
}
```

if 条件可以用bool值，比如

```go
if true {
	...
}
```

## **作用域**

---

??? note "例1. if的代码块是独立的作用域吗?"
	```go
	package main

	import "fmt"

	func main() {
		a := 1
		b := 2

		if 1 > 0 {
			fmt.Println("a:", a)
			fmt.Println("b:", b)
			a = 10
			b = 20
			a := 100
			b := 200
			fmt.Println("a:", a)
			fmt.Println("b:", b)
		}

		fmt.Println("a:", a)
		fmt.Println("b:", b)
	}
	```

	输出

	```text
	a: 1
	b: 2
	a: 100
	b: 200
	a: 10
	b: 20
	```

	答: if的代码块是独立的作用域

??? note "例2. if条件使用短声明时是独立的变量还是引用外部变量?"
	```go
	package main

	import "fmt"

	func main() {
		a := 1
		b := 2

		if a := 10000; 1 > 0 {
			fmt.Println("a:", a)
			fmt.Println("b:", b)
			a = 10
			b = 20
			a := 100
			b := 200
			fmt.Println("a:", a)
			fmt.Println("b:", b)
		}

		fmt.Println("a:", a)
		fmt.Println("b:", b)
	}
	```

	输出

	```text
	a: 10000
	b: 2
	a: 100
	b: 200
	a: 1
	b: 20
	```

	答: if条件使用短声明时是独立的变量。下面这个例子更能说明

	```go
	package main

	import "fmt"

	func main() {
		b := 2

		if a := 10000; 1 > 0 {
			fmt.Println("a:", a)
			fmt.Println("b:", b)
			a = 10
			b = 20
			a := 100
			b := 200
			fmt.Println("a:", a)
			fmt.Println("b:", b)
		}

		fmt.Println("a:", a)
		fmt.Println("b:", b)
	}
	```

	报错

	```text
	./if.go:19: undefined: a
	```

	在if外引用变量a，提示未找到，因为a只在if作用域

??? note "例3. if条件使用赋值语句时是独立的变量还是引用外部变量?"

	```go
	package main

	import "fmt"

	func main() {
		a := 1
		b := 2

		if a = 10000; 1 > 0 {
			fmt.Println("a:", a)
			fmt.Println("b:", b)
			a = 10
			b = 20
			a := 100
			b := 200
			fmt.Println("a:", a)
			fmt.Println("b:", b)
		}

		fmt.Println("a:", a)
		fmt.Println("b:", b)
	}
	```

	输出

	```text
	a: 10000
	b: 2
	a: 100
	b: 200
	a: 10
	b: 20
	```

	答: if条件使用赋值语句时是引用外部变量

??? note "例4. 若if条件不满足，那么条件句里的赋值语句是否有生效?"
	```go
	package main

	import "fmt"

	func main() {
		a := 1
		b := 2

		if a = 10000; false {
			fmt.Println("if-a:", a)
			fmt.Println("if-b:", b)
		}

		fmt.Println("a:", a)
		fmt.Println("b:", b)
	}
	```

	输出

	```text
	a: 10000
	b: 2
	```

	答: 若if条件不满足，那么条件句里的赋值语句依然生效。因此谨慎在if条件句里引用外部变量。
