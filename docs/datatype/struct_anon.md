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

!!! note ""
	除了基本类型，其他复合类型都可以用: T{name:value} 完成初始化。

	比如上面例子中的`struct{name string}`就是T，`{name: "hello"}`就是{name:value}
