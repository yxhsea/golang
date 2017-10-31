如果只是临时使用struct一次，而不是多次使用，用匿名struct即可

```go
package main

import "fmt"

func main() {
    a := struct{name string}{name: "hello"}
    fmt.Printf("%T\n", a)
    fmt.Printf("%v\n", a)
    fmt.Println(a.name)
}
```

输出

```text
struct { name string }
{hello}
hello
```
