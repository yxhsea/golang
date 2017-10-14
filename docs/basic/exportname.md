模块里名字第一个字符大写可以被其他模块引用，而如果首字符是小写则不行，比如`stringutil.go`内容:

```go
// Package stringutil contains utility functions for working with strings.
package stringutil

// Reverse returns its argument string reversed rune-wise left to right.
func reverse(s string) string {
        r := []rune(s)
        for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
                r[i], r[j] = r[j], r[i]
        }
        return string(r)
}
```

`hello.go`要引用这个模块：

```go
package main

import (
    "fmt"
    "github.com/cyent/repo1/stringutil"
)

func main() {
    fmt.Printf(stringutil.reverse("!oG ,olleH"))
}
```

这样`hello.go`编译会报错：

```text
# github.com/cyent/repo1/hello
./hello.go:10: cannot refer to unexported name stringutil.reverse
./hello.go:10: undefined: stringutil.reverse
```

正确应该是：

`stringutil.go`内容是：

```go
// Package stringutil contains utility functions for working with strings.
package stringutil

// Reverse returns its argument string reversed rune-wise left to right.
func Reverse(s string) string {
        r := []rune(s)
        for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
                r[i], r[j] = r[j], r[i]
        }
        return string(r)
}
```

`hello.go`要引用这个模块：

```go
package main

import (
    "fmt"
    "github.com/cyent/repo1/stringutil"
)

func main() {
    fmt.Printf(stringutil.Reverse("!oG ,olleH"))
}
```

这样编译就不会报错了
