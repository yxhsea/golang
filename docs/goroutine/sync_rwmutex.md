**读写锁: sync.RWMutex**

## **介绍**

---

读写锁是针对于读写操作的互斥锁。它与普通的互斥锁最大的不同就是，它可以分别针对读操作和写操作进行锁定和解锁操作。读写锁遵循的访问控制规则与互斥锁有所不同。在读写锁管辖的范围内，它允许任意个读操作的同时进行。但是，在同一时刻，它只允许有一个写操作在进行。并且，在某一个写操作被进行的过程中，读操作的进行也是不被允许的。也就是说，读写锁控制下的多个写操作之间都是互斥的，并且写操作与读操作之间也都是互斥的。但是，多个读操作之间却不存在互斥关系。

换句话说:

1. 同时只能有一个 goroutine 能够获得写锁定。

2. 同时可以有任意多个 gorouinte 获得读锁定。

3. 同时只能存在写锁定或读锁定（读和写互斥）。

## **方法**

---

```go
func (rw *RWMutex) Lock       //写锁定
func (rw *RWMutex) Unlock     //写解锁
func (rw *RWMutex) RLock      //读锁定
func (rw *RWMutex) RUnlock    //读解锁
```

Mutex和RWMutex都实现了Locker接口。该接口如下：

```go
type Locker interface {
    Lock()
    Unlock()
}
```

另外，还有一个内置方法

```go
func (rw *RWMutex) RLocker() Locker    //返回实现了sync.Locker接口的值
```

这个RLocker()作用是，使用Lock()和Unlock()来进行读锁定和读解锁，而无需RLock()和RUnlock()来进行读锁定和读解锁
