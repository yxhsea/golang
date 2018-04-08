## **查看CPU数量**

---

```text
runtime.NumCPU()
```

## **查看GOROOT**

---

```text
runtime.GOROOT()
```

## **查看操作系统**

---

```text
runtime.GOOS
```

输出有`linux`或`darwin`之类

## **设置最大进程数**

---

runtime.GOMAXPROCS(1) 设置为0时，等于没有做任何设置，即默认的本机cpu数量。当设置数量超过本机cpu时，则也为默认（即本机cpu数量）。因此可设置值的范围是1到本机cpu数量。这里的cpu数量都是指逻辑cpu，即核数，而非物理cpu数量

设置runtime.GOMAXPROCS()时候，go会stopTheWorld("GOMAXPROCS")，然后再startTheWorld()

## **查看有多少个协程**

---

```text
runtime.NumGoroutine()
```

例子

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func bar() {
    fmt.Println("bar:", runtime.NumGoroutine())
}

func foo() {
    fmt.Println("foo:", runtime.NumGoroutine())
    go bar()
    time.Sleep(100 * time.Millisecond)
}

func main() {
    fmt.Println("main:", runtime.NumGoroutine())
    go foo()
    time.Sleep(100 * time.Millisecond)
}
```

输出

```text
main: 1
foo: 2
bar: 3
```

可以看到：不开启协程时候默认就有一个协程
