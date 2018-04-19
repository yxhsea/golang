**互斥锁: sync.Mutex**

## **介绍**

---

用于主动控制Mutex类型的变量或者将Mutex类型作为struct的元素的变量在同一时间只被一个routine访问（即执行Lock()方法的代码块），这个Mutex带有2个方法：Lock()和Unlock()。互斥锁不区分读和写，即无论是print打印还是写操作都是互斥的

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var mutex sync.Mutex
	fmt.Printf("%+v\n", mutex)

	mutex.Lock()
	fmt.Printf("%+v\n", mutex)

	mutex.Unlock()
	fmt.Printf("%+v\n", mutex)
}
```

输出

```text
{state:0 sema:0}
{state:1 sema:0}
{state:0 sema:0}
```

可以看出当Lock()时，state为1，Unlock()时，state为0。

这段代码其实是为后面Lock()使用做了一个工作原理铺垫（个人猜测）：Lock()时候会判断state，若为0，则将state改为1；若为1，则阻塞等待，直到state变为0，然后再将state改为1。sync.Mutex是并发安全的。

## **使用**

---

下面这个例子是在官网例子上做了一点修改，更容易看出互斥效果

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string, id int) {
	c.mux.Lock()
	fmt.Printf("%d. Inc lock.\n", id)
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
	c.mux.Unlock()
	fmt.Printf("%d. Inc unlock.\n", id)
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	c.mux.Lock()
	fmt.Println("Value lock.")
	// Lock so only one goroutine at a time can access the map c.v.
	defer fmt.Println("Value unlock.")
	defer c.mux.Unlock()
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 10; i++ {
		go c.Inc("somekey", i)
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}
```

输出

```text
1. Inc lock.
1. Inc unlock.
0. Inc lock.
0. Inc unlock.
7. Inc lock.
7. Inc unlock.
8. Inc lock.
8. Inc unlock.
6. Inc lock.
6. Inc unlock.
3. Inc lock.
3. Inc unlock.
9. Inc lock.
9. Inc unlock.
2. Inc lock.
2. Inc unlock.
4. Inc lock.
4. Inc unlock.
5. Inc lock.
5. Inc unlock.
Value lock.
Value unlock.
10
```

- 每次输出结果也不太一样：0-9的顺序不一致

- 我们一般会在锁定互斥锁之后紧接着就用defer语句来保证该互斥锁的及时解锁，因为defer是先进后出

- Inc和Value方法的receiver是指针，这是必须的，因为Mutex锁对应的就是一个具体的变量（可以是struct，也可以是其他普通类型）

## **一个愿打一个愿挨**

---

互斥锁的使用是主动控制互斥锁，即一个愿打一个愿挨。比如即使一个routine里用了Lock()，但在另一个routine可以不理会这个锁就能访问这个struct，只需要不调用Lock()就行。例子如下

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

func (c *SafeCounter) Inc(key string) {
	c.mux.Lock()
	time.Sleep(3 * time.Second)
	c.v[key]++
	c.mux.Unlock()
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	go c.Inc("somekey")

	time.Sleep(1 * time.Second)
	c.v["somekey"]++
	fmt.Println(c.v["somekey"])

	time.Sleep(3 * time.Second)
	fmt.Println(c.v["somekey"])
}
```

输出

```text
1
2
```

`c.v["somekey"]++`之前，是已经锁了，本应该等2-3秒，才能输出，但是从程序开始运行，只过了1秒就输出了，说明Lock只是一种人为的互斥，是一种协议，并不是强制。

接下来看下不加锁的效果，即是否会产生冲突，例子如下

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string, id int) {
//	c.mux.Lock()
	fmt.Printf("%d. Inc lock.\n", id)
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
//	c.mux.Unlock()
	fmt.Printf("%d. Inc unlock.\n", id)
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	c.mux.Lock()
	fmt.Println("Value lock.")
	// Lock so only one goroutine at a time can access the map c.v.
	defer fmt.Println("Value unlock.")
	defer c.mux.Unlock()
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 2; i++ {
		go c.Inc("somekey", i)
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}
```

有时候输出

```text
0. Inc lock.
1. Inc lock.
0. Inc unlock.
1. Inc unlock.
Value lock.
Value unlock.
2
```

有时候输出

```text
1. Inc lock.
0. Inc lock.
fatal error: 1. Inc unlock.
concurrent map writes

goroutine 5 [running]:
runtime.throw(0x10b65f1, 0x15)
    /usr/local/go/src/runtime/panic.go:596 +0x95 fp=0xc420098e80 sp=0xc420098e60
runtime.mapassign(0x109d7a0, 0xc4200180f0, 0xc420098f90, 0x7)
    /usr/local/go/src/runtime/hashmap.go:499 +0x667 fp=0xc420098f20 sp=0xc420098e80
main.(*SafeCounter).Inc(0xc42000e270, 0x10b49f8, 0x7, 0x0)
    /Users/foo/test/b.go:20 +0x144 fp=0xc420098fc0 sp=0xc420098f20
runtime.goexit()
    /usr/local/go/src/runtime/asm_amd64.s:2197 +0x1 fp=0xc420098fc8 sp=0xc420098fc0
created by main.main
    /Users/foo/test/b.go:38 +0xcd

goroutine 1 [sleep]:
time.Sleep(0x3b9aca00)
    /usr/local/go/src/runtime/time.go:59 +0xf9
main.main()
    /Users/foo/test/b.go:41 +0xf2
```

这里报错是因为go语言的map不是并发安全，所以当并发不加锁时候就会存在冲突的可能。最新的go语言的sync包里新增了sync.Map，应该可以解决类似问题（还没研究过）

## **已经锁定的Mutex与特定的goroutine无关联**

---

已经锁定的Mutex并不与特定的goroutine相关联，这样可以利用一个goroutine对其加锁，再利用其他goroutine对其解锁，例子如下

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type MyStruct struct {
	v   int
	mux sync.Mutex
}

func (s *MyStruct) Lock() {
	s.mux.Lock()
}

func (s *MyStruct) Unlock() {
	s.mux.Unlock()
}

func main() {
	s := MyStruct{v: 0}
	s.v = 1
	fmt.Printf("%+v\n", s)

	go s.Lock()
	time.Sleep(1 * time.Second)
	fmt.Printf("%+v\n", s)

	go s.Unlock()
	time.Sleep(1 * time.Second)
	fmt.Printf("%+v\n", s)
}
```

输出

```text
{v:1 mux:{state:0 sema:0}}
{v:1 mux:{state:1 sema:0}}
{v:1 mux:{state:0 sema:0}}
```

可以看出，可以一个routine里锁定，另一个routine里解锁。因为锁只和具体变量关联，和routine无关，只要这个变量是共享的，比如通过指针传递，或者全局变量都可以。

虽然互斥锁可以被直接的在多个Goroutine之间共享，但是我们还是强烈建议把对同一个互斥锁的成对的锁定和解锁操作放在同一个层次的代码块中。例如，在同一个函数或方法中对某个互斥锁的进行锁定和解锁。
