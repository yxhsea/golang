type switches

```text
switch v := i.(type)  {
case int:
	...
case string:
	...
default:
	...
}
```

注意：之前类型断言是用i.($TYPE)比如i.(int)来判断是不是int类型，但是这里用关键字type。关键字type只能用在switch语句里，如果用在switch外面会报错，比如：

```text
a := i.(type)
fmt.Printf("%v %T\n", a, a)
```

报错：

```text
use of .(type) outside type switch
```

除了能识别内置类型，也可以识别其他类型，比如函数类型，或者带指针的struct，比如这个例子是带指针的struct

```go
type Foo interface {
    foo() int
}

type MyStruct struct {
    X, Y int
}

func (a *MyStruct) foo() int {
    return a.X + a.Y
}

func main() {
    var f Foo
    s := MyStruct{3, 4}
    f = &s
    fmt.Printf("%v,%T\n", f, f)
    switch v := f.(type) {
    case *MyStruct:
        fmt.Printf("1,%v,%T\n", v, v)
    default:
        fmt.Printf("2,%v,%T\n", v, v)
    }
}
```

输出

```text
&{3 4},*main.MyStruct
1,&{3 4},*main.MyStruct
```

注意：这个type用%T显示出来的是*main.MyStruct，而case里是*MyStruct，没有main喔。

这个例子是函数：

```go
func main() {
	pos := func () int { return 1 }
	fmt.Printf("%v,%T\n", pos, pos)

	var i interface{}
	i = pos
	fmt.Printf("%v,%T\n", i, i)

	switch v := i.(type) {
	case int:
		fmt.Printf("1,%v,%T\n", v, v)
	case func() int:
		fmt.Printf("2,%v,%T\n", v, v)
	case func(int) int:
		fmt.Printf("3,%v,%T\n", v, v)
	default:
		fmt.Printf("4,%v,%T\n", v, v)
	}
}
```

输出：

```text
0x1088f30,func() int
0x1088f30,func() int
2,0x1088f30,func() int
```

可以看出：case后面跟的就是i值的类型

注意：case后面不能跟不存在的自定义类型，比如：

```go
func main() {
    var i interface{}
    i = 1
    fmt.Printf("%v,%T\n", i, i)

    switch v := i.(type) {
    case int:
        fmt.Printf("1,%v,%T\n", v, v)
    case func() int:
        fmt.Printf("2,%v,%T\n", v, v)
    case func(int) int:
        fmt.Printf("3,%v,%T\n", v, v)
    case *MyStruct:
        fmt.Printf("4,%v,%T\n", v, v)
    default:
        fmt.Printf("5,%v,%T\n", v, v)
    }
}
```

报错：

```text
undefined: MyStruct
```
