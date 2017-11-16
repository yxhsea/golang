make：在初始化时候就指定len和cap

```go
a := make([]int, 5)  // len(a)=5
b := make([]int, 0, 5) // len(b)=0, cap(b)=5
b = b[:cap(b)] // len(b)=5, cap(b)=5
b = b[1:]      // len(b)=4, cap(b)=4
```

注意，在赋值时候，若超过len边界，则会报错，如

```go
package main

import "fmt"

func main() {
    sli1 := make([]int, 0, 5)
    fmt.Println(sli1, len(sli1), cap(sli1))

    sli1[0] = 3
    fmt.Println(sli1, len(sli1), cap(sli1))
}
```

报错

```text
[] 0 5
panic: runtime error: index out of range

goroutine 1 [running]:
main.main()
    /golang/example/slice.go:9 +0x2a3
exit status 2
```

改成如下才可以

```go
package main

import "fmt"

func main() {
    sli1 := make([]int, 0, 5)
    fmt.Println(sli1, len(sli1), cap(sli1))

    sli1 = sli1[:cap(sli1)]
    sli1[0] = 3
    fmt.Println(sli1, len(sli1), cap(sli1))
}
```

输出

```text
[] 0 5
[3 0 0 0 0] 5 5
```
