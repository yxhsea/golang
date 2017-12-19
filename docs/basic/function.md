## **入参**

---

函数可以接收0个或多个参数，参数需要指定类型，返回值也需要指定类型，比如

```go
func add(x int, y int) int {
	return x + y
}
```

上面表示声明一个函数add，接收2个参数，分别都是int类型，返回的类型也是int。

!!! warning ""
	1. main函数不用指定返回类型
	2. 函数里如果没有`return`，而是`fmt.Println`之类的，就不用指定返回类型

类型：类型跟在变量名后面，比如`#!go x int`，表示声明一个int类型的变量x

!!! note ""
	如果想了解为什么长这样，可以看 https://blog.golang.org/gos-declaration-syntax

!!! success "参数类型相同可以简写"
	如果声明的多个参数类型是相同的，可以只写最后一个参数类型，比如上面的`#!go x int, y int`也可以写成`#!go x, y int`

## **出参**

---

返回类型必须指定，比如本页最开始的例子中，如果改成

```go
func add(x int, y int) {
	return x + y
}
```

则报错:

```text
too many arguments to return
	have (int)
	want ()
```

多返回值：

```go
func swap(x, y string) (string, string)
```

返回值命名：

```go
func swap(x, y string) (a, b string)，这种通常用于return不跟参数
```

return不跟参数：

```go
func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}
```

上面return没有跟任何参数，实际等同于`return x, y`

即根据`func split(sum int) {==(x, y int)==}`顺序来定

!!! note ""
	这种return不跟参数方式不推荐，这种方式仅适用于比较短的函数，如果函数比较长，这样可读性就很不好了

## **入参和出参的变量不能重复声明**

---

这个例子是正确的

```go
package main

import "fmt"

func foo(a, b int) (c int) {
	return a + b
}

func main() {
	fmt.Println(foo(1, 2))
}
```

但如果改成

```go hl_lines="6"
package main

import "fmt"

func foo(a, b int) (c int) {
	a := 3
	return a + b
}

func main() {
	fmt.Println(foo(1, 2))
}
```

则报错

```text
./example.go:6: no new variables on left side of :=
```

下面这个例子依然是错误的

```go hl_lines="6"
package main

import "fmt"

func foo(a, b int) (c int) {
	c := 3
	return a + b
}

func main() {
	fmt.Println(foo(1, 2))
}
```

依然报错

```text
./example.go:6: no new variables on left side of :=
```
