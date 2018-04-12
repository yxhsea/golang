## **close**

---

```text
close(ch)
```

有且只有发送者可以关闭管道，接收者不能关闭管道

接收者可以通过`v, ok := <-ch`这种方式来测试管道是否关闭，若ok为false则表示管道已关闭

## **结合range**

---

可以用for range来循环取出管道里的数据，range当遇到管道关闭时候就会自动结束循环，例子：

```go
package main

import (
	"fmt"
)

func foo(c chan string) {
	c <- "a"
	c <- "b"
	close(c)
}

func main() {
	c := make(chan string)
	go foo(c)
	for i := range c {
		fmt.Println(i)
	}
}
```

输出

```text
a
b
```

> Closing is only necessary when the receiver must be told there are no more values coming, such as to terminate a range loop.

只有必须告知接收者没有数据要接收了才需要关闭管道，例如上面例子中需要告知range循环该结束了。如果不告知就会造成死锁

**生产者close()结合消费者<- 就可以实现类似多进程、多线程的wait、join这种等待效果**
