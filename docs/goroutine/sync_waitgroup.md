WaitGroup 用于等待一组 goroutine 结束，用法很简单。它有三个方法：

```text
func (wg *WaitGroup) Add(delta int)
func (wg *WaitGroup) Done()
func (wg *WaitGroup) Wait()
```

- Add 用来添加 goroutine 的个数

- Done 执行一次数量减 1

- Wait 用来等待结束

例子

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var wg sync.WaitGroup
	fmt.Printf("init:             %+v\n", wg)

	for i := 1; i < 10; i++ {
		// 计数加 1
		wg.Add(1)
		go func(i int) {
			fmt.Printf("goroutine%d start: %+v\n", i, wg)
			time.Sleep(11 * time.Second)
			// 计数减 1
			wg.Done()
			fmt.Printf("goroutine%d end:   %+v\n", i, wg)
		}(i)
		time.Sleep(time.Second)
	}

	// 等待执行结束
	wg.Wait()
	fmt.Printf("over:             %+v\n", wg)
}
```

输出

```text
init:             {noCopy:{} state1:[0 0 0 0 0 0 0 0 0 0 0 0] sema:0}
goroutine1 start: {noCopy:{} state1:[0 0 0 0 1 0 0 0 0 0 0 0] sema:0}
goroutine2 start: {noCopy:{} state1:[0 0 0 0 2 0 0 0 0 0 0 0] sema:0}
goroutine3 start: {noCopy:{} state1:[0 0 0 0 3 0 0 0 0 0 0 0] sema:0}
goroutine4 start: {noCopy:{} state1:[0 0 0 0 4 0 0 0 0 0 0 0] sema:0}
goroutine5 start: {noCopy:{} state1:[0 0 0 0 5 0 0 0 0 0 0 0] sema:0}
goroutine6 start: {noCopy:{} state1:[0 0 0 0 6 0 0 0 0 0 0 0] sema:0}
goroutine7 start: {noCopy:{} state1:[0 0 0 0 7 0 0 0 0 0 0 0] sema:0}
goroutine8 start: {noCopy:{} state1:[0 0 0 0 8 0 0 0 0 0 0 0] sema:0}
goroutine9 start: {noCopy:{} state1:[0 0 0 0 9 0 0 0 0 0 0 0] sema:0}
goroutine1 end:   {noCopy:{} state1:[1 0 0 0 8 0 0 0 0 0 0 0] sema:0}
goroutine2 end:   {noCopy:{} state1:[1 0 0 0 7 0 0 0 0 0 0 0] sema:0}
goroutine3 end:   {noCopy:{} state1:[1 0 0 0 6 0 0 0 0 0 0 0] sema:0}
goroutine4 end:   {noCopy:{} state1:[1 0 0 0 5 0 0 0 0 0 0 0] sema:0}
goroutine5 end:   {noCopy:{} state1:[1 0 0 0 4 0 0 0 0 0 0 0] sema:0}
goroutine6 end:   {noCopy:{} state1:[1 0 0 0 3 0 0 0 0 0 0 0] sema:0}
goroutine7 end:   {noCopy:{} state1:[1 0 0 0 2 0 0 0 0 0 0 0] sema:0}
goroutine8 end:   {noCopy:{} state1:[1 0 0 0 1 0 0 0 0 0 0 0] sema:0}
goroutine9 end:   {noCopy:{} state1:[0 0 0 0 0 0 0 0 0 0 0 0] sema:1}
over:             {noCopy:{} state1:[0 0 0 0 0 0 0 0 0 0 0 0] sema:0}
```

!!! warning
	wg.Add() 方法一定要在 goroutine 开始前执行
