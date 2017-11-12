## **接口是什么**

---

接口类型是由一组方法定义的集合。

接口类型的值可以存放实现这些方法的任何值。

## **接口和面向对象的关系**

---

golang中没有面向对象，但却可以实现面向对象的基础功能，类比如下：

- 基类：golang中的接口相当于面向对象中的基类（虽然接口并不完全是基类的概念，但在某些情况下，会把接口当作基类来使用），只不过继承基类需要显式，而golang的接口是隐式，并且只是方法名和传入传出参数相同即可

- 类：golang中的struct相当于面向对象中的类

- 类的属性：golang中struct的元素相当于面向对象中类的属性

- 类的方法：golang中struct的方法（就叫method，在某些语言里叫签名函数）相当于面向中的类的方法

例子详见[比较python和go的面向对象写法](/method/overview/#_2)

## **应用场景**

---

interface的使用要满足2个条件才有意义：

1. 实现了interface的几个struct是相似关系（比如docker和kvm都是虚拟机）、平级的，并且输入输出参数完全一致。（这点是interface的本质，能实现interface的肯定是满足这个条件）

2. 在业务逻辑上，调用实现interface的struct是不确定的，是通过某种方式传递进来，而不是顺序的业务逻辑，比如
这样是不对的，因为netns_struct、bridge_struct、veth_struct是有顺序的（先有netns、bridge，才能有veth）：

```go
func main() {
    var i vnic
    i = &netns_struct{...}
    i.Add()
    i = &bridge_struct{...}
    i.Add()
    i = &veth_struct{...}
    i.Add()
}
```

这样逻辑是正确的：

```go
var i vnic
switch opt {
case "netns":
    i = &netns.struct{}
case "bridge":
    i = &bridge.struct{}
case "veth":
    i = &veth.struct{}
}
i.Add()
i.Del()
```

就是说调用者这方对于实现interface的struct是根据某个参数（通过API传递过来，或者配置文件传递过来，或者etcd传递过来）来选择某个struct，这种逻辑才适用interface。而在vrouter程序逻辑里，netns、bridge、veth是被调用者依次执行，因此不适用interface。

总结适用interface的调用者业务逻辑（伪代码）：

```go
type I interface {
    ...
}

var i I
switch opt {    //opt通过某种方式传递进来，而不是写死
case "A":
    i = &structA{...}
case "B":
    i = &structB{...}
case "C":
    i = &structC{...}
default:
    errors.New("not support")
}
```

!!! note
	interface使用起来有无数种变形方式，但无论是那种，都要符合上面说的平行选一的业务逻辑。比如[golang-targz脑图](/attachment/golang_targz.itmz)（点击下载，并使用iThoughtsX打开），其中archive/tar/writer.go里

	```text
	func NewWriter(w io.Writer) *Writer { return &Writer{w: w} }
	```

	这里用`w io.Writer`就是因为有多种压缩方式，有gzip、有bzip2等等许多种，但无论哪种，都实现了io.Writer

## **interface语法**

---

有一个例子，如果不用interface：

```go
package main

import "fmt"

type MyStruct struct {
    X, Y int
}

func (a *MyStruct) foo() int {
    return a.X + a.Y
}

func main() {
    s := MyStruct{3, 4}
    fmt.Println(s.foo())
}
```

输出

```text
7
```

上面可以看到，可以用s.foo()这种方式来实现method。

现在用interface：

```go
package main

import "fmt"

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
    fmt.Println(f.foo())
}
```

输出

```text
7
```

## **传入和传出参数要带上**

---

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
