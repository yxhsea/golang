1. 不用interface时候

	```go
	package main

	import "fmt"

	type MyStruct struct {
	    X, Y int
	}

	func (a *MyStruct) add() int {
	    return a.X + a.Y
	}

	func main() {
	    s := MyStruct{3, 4}
	    fmt.Println(s.add())
	}
	```

	输出

	```text
	7
	```

2. 现在用interface

	```go hl_lines="5 6 7 18 19 20"
	package main

	import "fmt"

	type Adder interface {
	    add() int
	}

	type MyStruct struct {
	    X, Y int
	}

	func (a *MyStruct) add() int {
	    return a.X + a.Y
	}

	func main() {
	    var f Adder
	    s := MyStruct{3, 4}
	    f = &s
	    fmt.Println(f.add())
	}
	```

	上面黄色高亮部分就是接口部分

	!!! note "简写"
		```go
		var f Adder
		s := MyStruct{3, 4}
		f = &s
		```

		可以写成

		```go
		var f Adder = MyStruct{3, 4}
		```

	输出

	```text
	7
	```
