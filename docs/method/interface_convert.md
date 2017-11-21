```go hl_lines="19"
package main

import "fmt"

type Sumer interface {
	Sum()
}

type MyStruct struct {
	X, Y int
}

func (m *MyStruct) Sum() {
	fmt.Println(m.X + m.Y)
}

func main() {
	s := &MyStruct{3, 4}
	var i = Sumer(s)
	i.Sum()
}
```

输出

```text
7
```

!!! warning
	返回类型可以用来实现接口，但在内部并不等于类型转换：

	```go
	package main

	import "fmt"

	type Sumer interface {
		Sum()
	}

	type Fooer interface {
		Foo()
	}

	type MyStruct struct {
		X, Y int
	}

	func (m *MyStruct) Sum() {
		fmt.Println(m.X + m.Y)
	}

	func run(m *MyStruct) Sumer {
		return m
	}

	func run1(m *MyStruct) Fooer {
		return m
	}

	func main() {
		s := &MyStruct{3, 4}
		res := run(s)
		res.Sum()

		s1 := &MyStruct{3, 4}
		var i1 Sumer = s1
		i1.Sum()

		s2 := &MyStruct{3, 4}
		var i2 = Sumer(s2)
		i2.Sum()

		s3 := &MyStruct{3, 4}
		res3 := run1(s3)
		res3.Sum()
	}
	```

	报错：

	```text
	./example.go:26: cannot use m (type *MyStruct) as type Fooer in return argument:
		*MyStruct does not implement Fooer (missing Foo method)
	./example.go:44: res3.Sum undefined (type Fooer has no field or method Sum)
	```

	附类型转换的报错：
	```text
	./test.go:39: cannot convert s3 (type *MyStruct) to type Fooer:
	*MyStruct does not implement Fooer (missing Foo method)
	```

	比较上面2个错误可以发现报错内容是不一样的，虽然第二行的提示是一样的，都是没有实现，并且括号里说明了没有实现的原因，但第一行的前者是提示类型不对，后者是不能转换。这说明了返回类型可以用来实现接口，但在内部并不等于类型转换
