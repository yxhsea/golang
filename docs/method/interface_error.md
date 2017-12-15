## **error**

---

```go
package main

import (
    "fmt"
    "time"
)

type MyError struct {
    When time.Time
    What string
}

func (e *MyError) Error() string {
    return fmt.Sprintf("at %v, %s",
        e.When, e.What)
}

func main() {
    m := &MyError{time.Now(), "it didn't work"}

    var err error
    err = m
    fmt.Println(err)
}
```

输出

```text
at 2017-07-20 23:29:25.612912934 +0800 CST, it didn't work
```

可以看出，error和stringer功能相似

## **String()和Error()谁优先**

---

当String()和Error()同时存在的时候，Error()优先，例如：

```go
package main

import "fmt"

type MyStruct struct {
    X, Y int
}

func (s *MyStruct) String() string {
    return fmt.Sprintf("String-X: %v, Y: %v", s.X, s.Y)
}

func (s *MyStruct) Error() string {
    return fmt.Sprintf("Error-X: %v, Y: %v", s.X, s.Y)
}

func main() {
    s1 := &MyStruct{1, 2}
    fmt.Println(s1)
}
```

输出

```text
Error-X: 1, Y: 2
```

并不是因为Error()在代码中的位置靠下所以优先，若把上面String()和Error()对调，输出依然是Error

## **为什么要用error**

---

```go
package main

import (
	"fmt"
	"time"
)

type MyError struct {
	When time.Time
	What string
}

func (e *MyError) Error() string {
	return fmt.Sprintf("at %v, %s",
		e.When, e.What)
}

/*
func run() error {
	return &MyError{
		time.Now(),
		"it didn't work",
	}
}
*/

/*
func run() *MyError {
	return &MyError{
		time.Now(),
		"it didn't work",
	}
}
*/

/*
func run() interface{} {
	return &MyError{
		time.Now(),
		"it didn't work",
	}
}
*/

func main() {
	if err := run(); err != nil {
		fmt.Printf("%T\n", err)
		fmt.Println(err)
	}
}
```

上面3个run()任意取消一个注释，都能输出，并且输出都是：

```text
*main.MyError
at 2017-07-25 22:45:27.476615537 +0800 CST, it didn't work
```

问题：既然3个run()都能用，为什么要用error，而不是另外2个？

答：用哪个都可以，但既然有error，就用error吧，符合习惯，也为了让别人更容易看懂

因为这3个用哪个都是一模一样的，不仅最终输出一样，底层处理逻辑也是完全一样，因为fmt里有一段：

```go
switch verb {
case 'v', 's', 'x', 'X', 'q':
    // Is it an error or Stringer?
    // The duplication in the bodies is necessary:
    // setting handled and deferring catchPanic
    // must happen before calling the method.
    switch v := p.arg.(type) {
    case error:
        handled = true
        defer p.catchPanic(p.arg, verb)
        p.fmtString(v.Error(), verb)
        return

    case Stringer:
        handled = true
        defer p.catchPanic(p.arg, verb)
        p.fmtString(v.String(), verb)
        return
    }
}
```

说明会先判断下是否实现了error接口，即是否有Error()方法，如果有的话，就打印Error()。而根据接口的pair原理，无论是interface{}、struct、还是实现error接口，最终都会在fmt里再实现一次error接口，而上面例子中3种情况均实现error接口，所以完全一样，没有任何区别
