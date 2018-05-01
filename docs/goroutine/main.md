## **什么是协程**

---

关于协程本身的概念，详见[进程、线程、协程](/other/concurrent/#_5)

## **GO协程**

---

> goroutine（go协程）是由Go runtime管理的轻量级线程。

这句话表明了协程是用户态，因为是由Go runtime管理，而非OS内核管理

## **启动GO协程**

---

启动go协程方法：只需要在调用函数时候在前面加上go即可，如

```text
go say("hello")
```

## **单核和多核下，go协程的区别**

---

单核和多核情况下，go协程的执行结果是有差异的，看例子

同样的代码

```go
package main

import (
	"fmt"
	"time"
)

func say(s string) {
	for i := 0; i < 5; i++ {
		time.Sleep(100 * time.Millisecond)
		fmt.Println(s)
	}
}

func main() {
	go say("world")
	say("hello")
}
```

同样的执行

```text
for i in {1..10}; do
    go run goroutines.go | md5sum
done
```

在单核上的结果每次都一样

```text
0c2c59a85ba901ca20b4ff0935244b1c  -
0c2c59a85ba901ca20b4ff0935244b1c  -
0c2c59a85ba901ca20b4ff0935244b1c  -
0c2c59a85ba901ca20b4ff0935244b1c  -
0c2c59a85ba901ca20b4ff0935244b1c  -
0c2c59a85ba901ca20b4ff0935244b1c  -
0c2c59a85ba901ca20b4ff0935244b1c  -
0c2c59a85ba901ca20b4ff0935244b1c  -
0c2c59a85ba901ca20b4ff0935244b1c  -
0c2c59a85ba901ca20b4ff0935244b1c  -
```

在多核上的结果每次都不一样，这里仅贴出2次

```text
0c2c59a85ba901ca20b4ff0935244b1c  -
0c2c59a85ba901ca20b4ff0935244b1c  -
1b8ac04b5e950c4c30682a92a5dbd50e  -
f3b59387aa582b783ce750226e024610  -
0c2c59a85ba901ca20b4ff0935244b1c  -
1b8ac04b5e950c4c30682a92a5dbd50e  -
52eda0b32ac5579adf4cdb8fdcaaf9f7  -
0c2c59a85ba901ca20b4ff0935244b1c  -
0c2c59a85ba901ca20b4ff0935244b1c  -
77f2faf014eb84e230dad3d4c60e2380  -
```

```text
a5be9ea34f6d9aabf1bdb3ce106dc95d  -
0c2c59a85ba901ca20b4ff0935244b1c  -
a5be9ea34f6d9aabf1bdb3ce106dc95d  -
286dc19dc0c36ff4838e1e7492502b5d  -
0c2c59a85ba901ca20b4ff0935244b1c  -
f3b59387aa582b783ce750226e024610  -
a5be9ea34f6d9aabf1bdb3ce106dc95d  -
851344244bbb9b7e636252215dd88b1b  -
0c2c59a85ba901ca20b4ff0935244b1c  -
851344244bbb9b7e636252215dd88b1b  -
```

## **协程相关内置方法**

---

详见[runtime章节](/other/runtime/#_3)
