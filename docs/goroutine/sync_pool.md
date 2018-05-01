## **关于堆和栈的基础**

---

下面内容摘自 http://blog.cyeam.com/golang/2017/02/08/go-optimize-slice-pool

程序会从操作系统申请一块内存，而这块内存也会被分成堆和栈。栈可以简单得理解成一次函数调用内部申请到的内存，它们会随着函数的返回把内存还给系统。

```go
func F() {
    temp := make([]int, 0, 20)
    ...
}
```

类似于上面代码里面的temp变量，只是内函数内部申请的临时变量，并不会作为返回值返回，它就是被编译器申请到栈里面。**申请到栈内存好处：函数返回直接释放，不会引起垃圾回收，对性能没有影响。**

```go
func F() []int{
    a := make([]int, 0, 20)
    return a
}
```

而上面这段代码，申请的代码一模一样，但是申请后作为返回值返回了，编译器会认为变量之后还会被使用，当函数返回之后并不会将其内存归还，那么它就会被申请到堆上面了。**申请到堆上面的内存才会引起垃圾回收。**

```go
func F() {
    a := make([]int, 0, 20)
    b := make([]int, 0, 20000)
    l := 20
    c := make([]int, 0, l)
}
```

a和b代码一样，就是申请的空间不一样大，但是它们两个的命运是截然相反的。a前面已经介绍过，会申请到栈上面，而b，由于申请的内存较大，**编译器会把这种申请内存较大的变量转移到堆上面。即使是临时变量，申请过大也会在堆上面申请。**

而c，对我们而言其含义和a是一致的，**但是编译器对于这种不定长度的申请方式，也会在堆上面申请，即使申请的长度很短。**

实际项目基本都是通过c := make([]int, 0, l)来申请内存，长度都是不确定的。自然而然这些变量都会申请到堆上面了。Golang使用的垃圾回收算法是『标记——清除』。简单得说，就是程序要从操作系统申请一块比较大的内存，内存分成小块，通过链表链接。每次程序申请内存，就从链表上面遍历每一小块，找到符合的就返回其地址，没有合适的就从操作系统再申请。如果申请内存次数较多，而且申请的大小不固定，就会引起内存碎片化的问题。申请的堆内存并没有用完，但是用户申请的内存的时候却没有合适的空间提供。这样会遍历整个链表，还会继续向操作系统申请内存。这就能解释我一开始描述的问题，**申请一块内存变成了慢语句。**

**申请内存变成了慢语句，解决方法就是使用临时对象池**

## **使用**

---

> 该例子摘自https://www.jianshu.com/p/2bd41a8f2254

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// 一个[]byte的对象池，每个对象为一个[]byte
var bytePool = sync.Pool{
	New: func() interface{} {
		b := make([]byte, 1024)
		return &b
	},
}

func main() {
	a := time.Now().Unix()
	// 不使用对象池
	for i := 0; i < 1000000000; i++ {
		obj := make([]byte, 1024)
		_ = obj
	}
	b := time.Now().Unix()
	// 使用对象池
	for i := 0; i < 1000000000; i++ {
		obj := bytePool.Get().(*[]byte)
		bytePool.Put(obj)
	}
	c := time.Now().Unix()
	fmt.Println("without pool ", b-a, "s")
	fmt.Println("with    pool ", c-b, "s")
}
```

输出

```text
without pool  20 s
with    pool  15 s
```

## **详解**

---

```go
package main

import (
	"fmt"
	"sync"
)

// 一个[]byte的对象池，每个对象为一个[]byte
var bytePool = sync.Pool{
	New: func() interface{} {
		b := make([]byte, 8)
		return &b
	},
}

func main() {
	fmt.Printf("%T\n", bytePool)
	fmt.Printf("%+v\n", bytePool)

	obj := bytePool.Get().(*[]byte)

	fmt.Printf("%T\n", obj)
	fmt.Printf("%v\n", obj)
}
```

输出

```text
sync.Pool
{noCopy:{} local:<nil> localSize:0 New:0x1090180}
*[]uint8
&[0 0 0 0 0 0 0 0]
```

## **为什么要用类型断言**

---

下面通过3个例子来说明为什么要用类型断言`(*int)`

首先，这种用法是不对的

```go
package main

import "fmt"

type MyStruct struct {
	e func() interface{}
}

var foo = MyStruct{
	e: func() interface{} {
		i := 1
		return &i
	},
}

func main() {
	fmt.Printf("foo: %T\n", foo)
	fmt.Printf("foo: %+v\n", foo)

	a := foo.e()
	fmt.Printf("a: %T\n", a)
	fmt.Printf("a: %+v\n", a)

	a = 2
	fmt.Printf("a: %T\n", a)
	fmt.Printf("a: %+v\n", a)
}
```

输出

```text
foo: main.MyStruct
foo: {e:0x10901f0}
a: *int
a: 0xc4200160b0
a: int
a: 2
```

可以看到，最后a的类型是int，而不是期望中的*int。为什么会这样，因为a是一个空接口interface{}，因此a可以接受任何类型的值

那么改造一下（这样也是错的）：

```go
package main

import "fmt"

type MyStruct struct {
	e func() interface{}
}

var foo = MyStruct{
	e: func() interface{} {
		i := 1
		return &i
	},
}

func main() {
	fmt.Printf("foo: %T\n", foo)
	fmt.Printf("foo: %+v\n", foo)

	a := foo.e()
	fmt.Printf("a: %T\n", a)
	fmt.Printf("a: %+v\n", a)

	*a = 2
	fmt.Printf("a: %T\n", a)
	fmt.Printf("a: %+v\n", a)
}
```

报错

```text
invalid indirect of a (type interface {})
```

因为接口类型（a是空接口）是不能使用`*a`来做赋值的。接下来，使用类型断言

```go
package main

import "fmt"

type MyStruct struct {
	e func() interface{}
}

var foo = MyStruct{
	e: func() interface{} {
		i := 1
		return &i
	},
}

func main() {
	fmt.Printf("foo: %T\n", foo)
	fmt.Printf("foo: %+v\n", foo)

	a := foo.e().(*int)
	fmt.Printf("a: %T\n", a)
	fmt.Printf("a: %+v\n", a)

	*a = 2
	fmt.Printf("a: %T\n", a)
	fmt.Printf("a: %+v\n", a)
}
```

输出

```text
foo: main.MyStruct
foo: {e:0x1090210}
a: *int
a: 0xc4200160b0
a: *int
a: 0xc4200160b0
```

这样才是预期的，即a依然为指针

## **pool是飞机，起步加速不如跑车**

---

**只有当每个对象占用内存较大时候，用pool才会改善性能**

对比1（起步阶段）:

```go hl_lines="12"
package main

import (
	"fmt"
	"sync"
	"time"
)

// 一个[]byte的对象池，每个对象为一个[]byte
var bytePool = sync.Pool{
	New: func() interface{} {
		b := make([]byte, 1)
		return &b
	},
}

func main() {
	a := time.Now().Unix()
	// 不使用对象池
	for i := 0; i < 1000000000; i++ {
		obj := make([]byte, 1)
		_ = obj
	}
	b := time.Now().Unix()
	// 使用对象池
	for i := 0; i < 1000000000; i++ {
		obj := bytePool.Get().(*[]byte)
		bytePool.Put(obj)
	}
	c := time.Now().Unix()
	fmt.Println("without pool ", b-a, "s")
	fmt.Println("with    pool ", c-b, "s")
}
```

输出

```text
without pool  0 s
with    pool  17 s
```

可以看到，当[]byte只有1个元素时候，用pool性能反而更差

对比2（追赶阶段）:

```go hl_lines="12"
package main

import (
	"fmt"
	"sync"
	"time"
)

// 一个[]byte的对象池，每个对象为一个[]byte
var bytePool = sync.Pool{
	New: func() interface{} {
		b := make([]byte, 800)
		return &b
	},
}

func main() {
	a := time.Now().Unix()
	// 不使用对象池
	for i := 0; i < 1000000000; i++ {
		obj := make([]byte, 800)
		_ = obj
	}
	b := time.Now().Unix()
	// 使用对象池
	for i := 0; i < 1000000000; i++ {
		obj := bytePool.Get().(*[]byte)
		bytePool.Put(obj)
	}
	c := time.Now().Unix()
	fmt.Println("without pool ", b-a, "s")
	fmt.Println("with    pool ", c-b, "s")
}
```

输出

```text
without pool  16 s
with    pool  17 s
```

可以看到，飞机快赶上跑车了

对比3（超越阶段）:

```go hl_lines="12"
package main

import (
	"fmt"
	"sync"
	"time"
)

// 一个[]byte的对象池，每个对象为一个[]byte
var bytePool = sync.Pool{
	New: func() interface{} {
		b := make([]byte, 8000)
		return &b
	},
}

func main() {
	a := time.Now().Unix()
	// 不使用对象池
	for i := 0; i < 1000000000; i++ {
		obj := make([]byte, 8000)
		_ = obj
	}
	b := time.Now().Unix()
	// 使用对象池
	for i := 0; i < 1000000000; i++ {
		obj := bytePool.Get().(*[]byte)
		bytePool.Put(obj)
	}
	c := time.Now().Unix()
	fmt.Println("without pool ", b-a, "s")
	fmt.Println("with    pool ", c-b, "s")
}
```

输出

```text
without pool  128 s
with    pool  17 s
```

!!! note "可以看到2个特征"
	1. 当每个对象的内存小于一定量的时候，不使用pool的性能秒杀使用pool；当内存处于某个量的时候，不使用pool和使用pool性能相当；当内存大于某个量的时候，使用pool的优势就显现出来了

	2. 不使用pool，那么对象占用内存越大，性能下降越厉害；使用pool，无论对象占用内存大还是小，性能都保持不变。可以看到pool有点像飞机，虽然起步比跑车慢，但后劲十足。

	即：pool适合占用内存大且并发量大的场景。当内存小并发量少的时候，使用pool适得其反
