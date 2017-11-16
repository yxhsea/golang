nil interface的官方解释：Interface values with nil underlying values

只有声明了但没赋值的interface才是nil interface，只要赋值了，即使赋了一个nil类型，也不是nil interface了

> Calling a method on a nil interface is a run-time error because there is no type inside the interface tuple to indicate which concrete method to call.

nil interface是不能调用方法的，如果调用方法，比如xxx是个nil interface，那么xxx.M()会runtime error。

另外，如果interface的值是空的，那么这个interface的值类型就是<nil>，这时候.method()来调用方法的话，传递进去的receiver的值类型也将是nil

注意，这种情况下，如果method里再调用，就会抛出运行时异常

举例：

```go
package main

import "fmt"

type I interface {
	M()
}

type T struct {
	S string
}

func (t *T) M() {
	if t == nil {
		fmt.Println("<nil>")
		return
	}
	fmt.Println(t.S)
}

func main() {
	var i I

	var t *T
	i = t
	describe(i)
	i.M()

	i = &T{"hello"}
	describe(i)
	i.M()
}

func describe(i I) {
	fmt.Printf("(%v, %T)\n", i, i)
}
```

输出

```text
(<nil>, *main.T)
<nil>
(&{hello}, *main.T)
hello
```

如果把上面M()里面的return这行注释掉，则会输出

```text
(<nil>, *main.T)
<nil>
panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0xffffffff addr=0x0 pc=0xc8971]

goroutine 1 [running]:
main.(*T).M(0x0, 0x0)
	/tmp/sandbox104196984/main.go:18 +0x31
main.main()
	/tmp/sandbox104196984/main.go:27 +0x60
```

> If the concrete value inside the interface itself is nil, the method will be called with a nil receiver.

> In some languages this would trigger a null pointer exception, but in Go it is common to write methods that gracefully handle being called with a nil receiver (as with the method M in this example.)

> Note that an interface value that holds a nil concrete value is itself non-nil.

上面是官网的话，其中最后一句，是说当声明一个interface时候，这个interface是nil，然后给这个interface赋了一个为nil的变量时候，这个interface就不是nil。测试例子：

```go
package main

import "fmt"

type I interface {
    M()
}

type T []int

func (t T) M() {
    fmt.Printf("%T\n", t)
    if t == nil {
        fmt.Println("<nil>")
        return
    }
}

func main() {
    var a T
    fmt.Printf("%T %v\n", a, a)
    if a == nil {
        fmt.Println("a is nil")
    }

    var i I
    if i == nil {
        fmt.Println("i is nil")
    }

    i = a
    if i == nil {
        fmt.Println("i is nil also.")
    }
}
```

输出

```text
main.T []
a is nil
i is nil
```
