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
