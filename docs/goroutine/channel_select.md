## **channel结合select和case**

---

select和case的组合可以使哪个管道就绪（对端已阻塞），就读取该管道数据并执行相应case的代码块。

官网译: select 会阻塞，直到条件分支中的某个可以继续执行，这时就会执行那个条件分支。当多个都准备好的时候，会随机选择一个。

```go
package main

import (
	"fmt"
)

func receive(ch1, ch2, ch3, quit chan int) {
	for i := 0; i < 2; i++ {
		fmt.Printf("receive %d from ch1\n", <-ch1)
		fmt.Printf("receive %d from ch2\n", <-ch2)
		fmt.Printf("receive %d from ch3\n", <-ch3)
	}
	quit <- 0
}

func send(ch1, ch2, ch3, quit chan int) {
	for i := 0; i < 10; i++ {
		select {
		case ch1 <- i:
			fmt.Printf("send %d to ch1\n", i)
		case ch2 <- i:
			fmt.Printf("send %d to ch2\n", i)
		case ch3 <- i:
			fmt.Printf("send %d to ch3\n", i)
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	ch3 := make(chan int)
	quit := make(chan int)
	go receive(ch1, ch2, ch3, quit)
	send(ch1, ch2, ch3, quit)
}
```

输出：输出不固定，因为是并行的，大概100次执行结果中有3-7次和其他不一样，大部分结果是

```text
send 0 to ch1
receive 0 from ch1
receive 1 from ch2
send 1 to ch2
send 2 to ch3
receive 2 from ch3
receive 3 from ch1
send 3 to ch1
send 4 to ch2
receive 4 from ch2
receive 5 from ch3
send 5 to ch3
quit
```

## **select的随机性**

---

```go
package main

import (
	"fmt"
	"time"
)

func receive(ch chan int) {
	for {
		<-ch
	}
}

func send(ch1, ch2, ch3 chan int) {
	for i := 0; i < 10; i++ {
		// sleep是为了保证所有的管道receiver都已阻塞等待数据
		time.Sleep(1000 * time.Millisecond)
		select {
		case ch1 <- i:
			fmt.Printf("send %d to ch1\n", i)
		case ch2 <- i:
			fmt.Printf("send %d to ch2\n", i)
		case ch3 <- i:
			fmt.Printf("send %d to ch3\n", i)
		}
	}
}

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	ch3 := make(chan int)
	go receive(ch1)
	go receive(ch2)
	go receive(ch3)
	send(ch1, ch2, ch3)
}
```

每次输出都不一样，且找不到规律，说明是随机性的

第一次

```text
send 0 to ch2
send 1 to ch2
send 2 to ch1
send 3 to ch3
send 4 to ch2
send 5 to ch2
send 6 to ch1
send 7 to ch2
send 8 to ch3
send 9 to ch1
```

第二次

```text
send 0 to ch3
send 1 to ch1
send 2 to ch1
send 3 to ch1
send 4 to ch2
send 5 to ch2
send 6 to ch2
send 7 to ch1
send 8 to ch1
send 9 to ch3
```

第三次

```text
send 0 to ch1
send 1 to ch1
send 2 to ch1
send 3 to ch3
send 4 to ch1
send 5 to ch2
send 6 to ch2
send 7 to ch2
send 8 to ch2
send 9 to ch1
```

## **select支持default**

---

当所有case都阻塞的时候，执行default

```go
select {
case ch1 <- i:
	fmt.Printf("send %d to ch1\n", i)
case ch2 <- i:
	fmt.Printf("send %d to ch2\n", i)
case ch3 <- i:
	fmt.Printf("send %d to ch3\n", i)
default:
    fmt.Println("default")
}
```

可以在channel里使用select的default来告知channe已满
