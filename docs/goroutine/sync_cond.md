## **用途**

---

与互斥量不同，条件变量的作用并不是保证在同一时刻仅有一个线程访问某一个共享数据，而是在对应的共享数据的状态发生变化时，通知其他因此而被阻塞的线程。条件变量总是与互斥量组合使用。互斥量为共享数据的访问提供互斥支持，而条件变量可以就共享数据的状态的变化向相关线程发出通知。

## **声明**

---

```text
lock := new(sync.Mutex)
cond := sync.NewCond(lock)
```

也可以写成一行

```text
cond := sync.NewCond(new(sync.Mutex))
```

## **方法**

---

```
cond.L.Lock()
cond.L.Unlock()
cond.Wait()
cond.Signal()
cond.Broadcast()
```

!!! note
	- `cond.L.Lock()`和`cond.L.Unlock()`：也可以使用`lock.Lock()`和`lock.Unlock()`，完全一样，因为是指针转递

	- `cond.Wait()`：Unlock()->***阻塞等待通知(即等待Signal()或Broadcast()的通知)->收到通知***->Lock()

	- `cond.Signal()`：通知一个Wait()了的，若没有Wait()，也不会报错。**Signal()通知的顺序是根据原来加入通知列表(Wait())的先入先出**

	- `cond.Broadcast()`: 通知所有Wait()了的，若没有Wait()，也不会报错

## **工作原理(摘取)**

---

下面图1-图3来自"郝林"的go语言书籍

没用条件变量时候:

![](/img/cond_1.png)

图1：生产者线程添加数据库的流程

图1存在一个问题：当队列总是满的时候，就会不停的循环获取队列状态，因此也不会释放锁，而消费者就无法获得锁来取走队列中的数据。如果队列中的数据无法取走，那么队列就永远都是满的，导致了死锁。

使用条件变量后，解决了这个问题：

![](/img/cond_2.png)

图2：生产者线程添加数据库的流程

图2解决了图1的问题：当队列满的时候，则先解锁，这样就不会死锁（释放锁给其他人用）

附：和图2匹配的消费者线程

![](/img/cond_3.png)

图3：消费者线程

## **工作原理(原创)**

---

![](/img/cond_4.png)

抢占锁后，判断自己是否满足处理 ***条件***，如果不满足则先 ***释放锁给别人用***，然后自己 ***阻塞***，等别人主动 ***通知自己***(肯定是别人的事情做完了才会通知自己)，就解除阻塞，然后再去 ***获得锁***

!!! error
	该图逻辑也许一开始看过去没啥问题，但是实际执行起来，就会有一些异常情况，因此建议能用CSP模型就用（即channel），尽量不使用锁

## **示例**

---

```go
package main

import (
	"fmt"
	"sync"
	"time"
)


func main() {
	cond := sync.NewCond(new(sync.Mutex))
	condition := 0

	// Consumer
	go func() {
		for {
			cond.L.Lock()
			for condition == 0 {
				cond.Wait()
			}
			condition--
			fmt.Printf("Consumer: %d\n", condition)
			cond.Signal()
			cond.L.Unlock()
		}
	}()

	// Producer
	for {
		time.Sleep(time.Second)
		cond.L.Lock()
		for condition == 3 {
			cond.Wait()
		}
		condition++
		fmt.Printf("Producer: %d\n", condition)
		cond.Signal()
		cond.L.Unlock()
	}
}
```

输出

```text
Producer: 1
Consumer: 0
Producer: 1
Consumer: 0
Producer: 1
Consumer: 0
Producer: 1
Consumer: 0
Producer: 1
Consumer: 0
...
```

注意：**这个例子看过去挺好，但只能用于单生产者+单消费者，同时对condition的判断只有0和1这种布尔值状态**

实际使用中，应当遵循先channel后锁的顺序，即channel如果能满足需求，则不要用锁，如果场景比较复杂，channel无法满足，再加上锁来控制。因为channel本身就是先进先出，等同于消息队列

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan int, 3)
	v := 0

	// Consumer
	go func() {
		for {
			fmt.Printf("Consumer: %d\n", <-ch)
		}
	}()

	// Producer
	for {
		v++
		fmt.Printf("Producer: %d\n", v)
		ch <- v
		time.Sleep(time.Second)
	}
}
```

```text
Producer: 1
Consumer: 1
Producer: 2
Consumer: 2
Producer: 3
Consumer: 3
Producer: 4
Consumer: 4
Producer: 5
Consumer: 5
...
```

## **条件变量和sync.Cond关系**

---

**以下均为个人理解（不一定准确）**

条件变量是一种应用场景，不是只有golang才有条件变量，其他操作系统、语言也都有条件变量

sync.Cond是golang为了适配这种应用场景而开发出来的配套组件。准确说是通过sync.Cond配合sync.Mutex来实现条件变量的应用场景。

因此，sync.Cond本身并不是什么条件变量，只是它的方法可以让你实现条件变量的应用，所以干脆名字就叫条件变量。也就是说，完全可以不用于条件变量而用于其他环境，只是没人这么干而已，而且也不是初衷。

## **并发不容易**

---

条件变量只是解决了互斥量的问题，或者说配合互斥量解决了锁的问题，但是并不能实现并发，或者说实现并发要对锁有很好的设计和控制，否则容易死锁或非预期的情况出现。

因为不知道锁什么时候被非预期的协程给抢占了，导致死锁或非预期的情况发生。

即如果把上面原创的工作原理图里再加上一个生产者或消费者，就有可能导致非预期情况发生，比如Signal时候锁会被Wait的协程给获得(假设Signal后是Unlock)，但因为不止1个并发，结果被另一个并发的协程给抢占了锁，而且是在非同一代码状态下（比如被另一个协程的第一个Lock）

条件变量因为在中间又加了一个释放锁、获取锁，因此会把情况搞得复杂，一个生产一个消费就不容易了，当大于2个协程时候就容易出现异常情况。也就是说能不用条件变量尽量不用，免得搞的太复杂不容易排查。

## **channel和sync区别**

---

一般并发就是指多个程序逻辑上同时运行，但肯定会遇到一些非线程安全的东西需要做同步（无法并行，只能串行），这个时候就有sync和channel2种方式实现同步。只不过channel除了做同步，还能用于传递变量（即通过通信来共享变量），而sync只有同步作用（通过共享变量来通信）
