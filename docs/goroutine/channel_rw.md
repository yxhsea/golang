channel支持3种类型（通过%T看到的）：

- 读写类型：`chan int`

- 只读类型：`<-chan int`，叫做receive-only

- 只写类型：`chan<- int`，叫做send-only

通过函数参数传递时候，若原来是读写，则可以转成只读或只写，但如果已经是只读或只写，则只能保持类型，无法转为其他类型（比如原来是只读，则只能是只读，无法转成只写，也无法转为读写），例子如下：

```go
package main

import (
	"fmt"
	"time"
)

func send(c chan<- int) {
	fmt.Printf("send: %T\n", c)
	c <- 1
}

func recv(c <-chan int) {
	fmt.Printf("recv: %T\n", c)
	fmt.Println(<-c)
}

func main() {
	c := make(chan int)
	fmt.Printf("%T\n", c)
	go send(c)
	go recv(c)
	time.Sleep(1 * time.Second)
}
```

输出

```text
chan int
send: chan<- int
recv: <-chan int
1
```
