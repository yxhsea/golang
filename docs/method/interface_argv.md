如果实现接口的签名函数有传入参数或传出参数，则接口定义时候要带上，比如：

```go
package main

import "fmt"

type Adder interface {
    Add(x, y int) (a int)
}

type MyStruct struct {
    X, Y int
}

func (m *MyStruct) Add(a, b int) int {
    return a + b + m.X + m.Y
}

func main() {
    var a Adder
    a = &MyStruct{3, 4}
    res := a.Add(5, 6)
    fmt.Println(res)
}
```

输出

```text
18
```

!!! note
	上面的

	```go
	type Adder interface {
	    Add(x, y int) (a int)
	}
	```

	可以改为

	```go
	type Adder interface {
	    Add(int, int) (a int)
	}
	```

	也可以改为

	```go
	type Adder interface {
	    Add(a,b int) (x int)
	}
	```

	也可以改为

	```go
	type Adder interface {
	    Add(a, b int) (int)
	}
	```

	但是输入和输出参数不能存在相同变量名，比如下面这样就会报错：

	```go
	type Adder interface {
	    Add(x, y int) (x int)
	}
	```

	报错

	```text
	duplicate argument x
	```
