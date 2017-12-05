method就是函数，只不过拥有receiver参数（普通函数没有receiver参数）

recevier写在func和method名字之间，比如：

```go hl_lines="5"
type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	a := Vertex{3, 4}
	fmt.Println(a.Abs())
}
```

!!! note
	上面v Vertex就是recevier

	高亮的这行代码可以解读为：Abs method拥有一个类型为Vertex，名字为v的receiver

注意：

1. 内置type不能作为receiver

	```go
	func (a int) foo() int {
	    return a + 1
	}

	func main() {
	    i := 1
	    f := i.foo()
	    fmt.Println(a)
	}
	```

	报错

	```text
	cannot define new methods on non-local type int
	```

2. 同一个包里自定义的type，才能作为method的receiver

	$GOPATH/src/github.com/cyent/golang/example/foo.go

	```go
	package foo

	type MyInt int
	```

	bar.go

	```go
	package main

	import (
	    "fmt"
	    "github.com/cyent/golang/example/foo"
	)

	func main() {
	    a := foo.MyInt(1)
	    fmt.Println(a)
	}
	```

	输出

	```text
	1
	```

	但如果把bar.go改成

	```go
	package main

	import (
	    "fmt"
	    "github.com/cyent/golang/example/foo"
	)

	func (x foo.MyInt) mymethod() int {
	    return int(x+1)
	}

	func main() {
	    a := foo.MyInt(1)
	    fmt.Println(a)

	    b := a.mymethod()
	    fmt.Println(b)
	}
	```

	报错

	```text
	./bar.go:8: cannot define new methods on non-local type foo.MyInt
	./bar.go:16: a.mymethod undefined (type foo.MyInt has no field or method mymethod)
	```

	上面这个例子说明了导入的包里的自定义type可以用（type首字母大写），但不能作为method的receiver

3. 只有放在包级别的type，并且首字母大写的，才能被导入给其他模块引用，详见[数据类型章节-自定义类型](/basic/type_custom/#_3)的第4点
