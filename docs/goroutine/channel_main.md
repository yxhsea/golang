## **channel**

---

channel是有类型的管道，可以用channel操作符 `<-` 对其发送或者接收值。

## **创建channel**

---

```text
c := make(chan int)
```

注意：管道必须指定类型，比如这里是int，即往管道里传送的数据只能是int类型。当然可以是interface{}这种空接口方式，这样就可以传送各种数据类型了

也可以用var来声明管道，但是好像没啥意义

```go
package main

import "fmt"


func main() {
	var c chan int
	fmt.Printf("%T %v\n", c, c)
	d := make(chan int)
	fmt.Printf("%T %v\n", d, d)
}
```

输出

```text
chan int <nil>
chan int 0xc42006e060
```

## **管道方向**

---

```text
ch <- v // Send v to channel ch.
v := <-ch // Receive from ch, and
           // assign value to v.
```

“箭头”就是数据流的方向

**注意接受时候<-和chan变量之间没有空格，虽然有空格也不会报错，但是ide会提示，因此还是依照规范不空格比较好**

## **基本使用**

---

```go
package main

import "fmt"

func foo(i int, c chan int) {
    c <- i
    fmt.Println("send:", i)
}

func main() {
    c := make(chan int)
    go foo(0, c)
    res := <-c
    fmt.Println("receive:", res)
}
```

输出

```text
send: 0
receive: 0
```

默认情况下，在另一端准备好之前，发送和接收都会阻塞。这使得 goroutine 可以在没有明确的锁或竞态变量的情况下进行同步。

发动和接收数据应当在并行线上，而不能是串行的，因为发送和接收都会阻塞，如果串行，就会死锁（就是一个一直阻塞在那等对端），但不用为此操心，因为go在执行时候（编译会通过）会报错，比如：

```go
package main

func main() {
	c := make(chan int)
	c <- 0
	<-c
}
```

报错：

```text
fatal error: all goroutines are asleep - deadlock!
```
