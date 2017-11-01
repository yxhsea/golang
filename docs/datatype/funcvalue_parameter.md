一个函数可以作为参数传递给另一个函数：

```go
package main

import "fmt"

func dd(i func(int, int) int) int {
    fmt.Printf("i type: %T\n", i)
    return i(1, 2)
}

func main() {
    ee := func(x, y int) int {
        return x + y
    }
    fmt.Printf("ee type: %T\n", ee)
    fmt.Println(dd(ee))
}
```

输出

```text
ee type: func(int, int) int
i type: func(int, int) int
3
```

上面这个例子就是函数ee作为参数传递给函数dd

!!! note
	注意：为什么dd(ee)这样是可以的，是因为ee的值的类型和dd的参数i的类型是一致的

	上面这种做法，也可以这么去理解：

	比如简单的函数：

	```go
	func abc(i int) int {
	    return i + 1
	}
	func main() {
	    fmt.Println(abc(2))
	}
	```

	上面可以理解为2的类型为int，和abc的参数i的类型一致，所以可以abc(2)，接着就是执行abc函数，将2赋值给i，然后执行2 + 1，结果为3

	同理，在上面的上面这个例子中，ee的类型为func(int, int) int，和dd的参数i的类型一致，所以可以dd(ee)，接着就是执行dd函数，将ee赋值给i，然后执行i(1, 2)，这里有点像变量替换，即i(1, 2)可以替换成ee(1, 2)，即return ee(1, 2)，也就是return 1 + 2，结果为3。（这行描述不一定对，只是目前的理解，也可能不是把ee带进去，而是ee的函数本身直接带进去）

包级别的也能用函数值方式来声明函数，只不过要用var：

```go
var abc = func (i int) int {
    return i + 1
}
func main() {
    fmt.Println(abc(2))
}
```

当然也可以闲的蛋疼完整写法：

```go
var abc func (int) int = func (i int) int {
    return i + 1
}
func main() {
    fmt.Println(abc(2))
}
```
