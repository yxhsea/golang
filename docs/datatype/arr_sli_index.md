## **startIndex和endIndex**

---

[startIndex:endIndex]

表示的是从第startIndex下标的元素开始，直到endIndex-1下标的元素结束。

其中元素下标是从0开始计数。如果用通俗的话来记忆就是从第startIndex+1开始，到第endIndex结束。

注意：这里的endIndex是指可见元素到哪为止，即影响len()，而不影响cap()，其中cap()都是到最后一个元素

## **下标举例说明**

---

- [1:5] 从第二个元素开始（0为第一个元素，1为第二个元素），取到第5个元素（下标为5-1=4，下标4表示第5个元素）出来，同时删除第一个元素，但cap依然是到最后一个元素

- [:] 取所有元素，没有删除元素

- [1:] 从第二个元素开始取所有元素，同时删除第一个元素

- [:5] 等同于[0:5]，即从第一个元素开始，取到第5个元素出来，但cap依然是到最后一个元素

## **下标边界写法**

---

首先说下，边界这2个字是我自己取的

若声明一个变量`var a [10]int`，则下面的语句是相同的

```text
a[0:10]
a[:10]
a[0:]
a[:]
a[:cap(a)]
```

举例如下:

```go
package main

import "fmt"

func main() {
    arr := []int{2, 3, 5, 7, 11, 13}
    sli1 := arr[1:4]
    fmt.Println("[1:4]", sli1)
    fmt.Println("[1:4]", len(sli1))
    fmt.Println("[1:4]", cap(sli1))
    fmt.Println("-----")
    sli2 := arr[1:3]
    fmt.Println("[1:3]", sli2)
    fmt.Println("[1:3]", len(sli2))
    fmt.Println("[1:3]", cap(sli2))
    fmt.Println("-----")
    sli3 := arr[1:2]
    fmt.Println("[1:2]", sli3)
    fmt.Println("[1:2]", len(sli3))
    fmt.Println("[1:2]", cap(sli3))
    fmt.Println("-----")
    sli4 := arr[1:1]
    fmt.Println("[1:1]", sli4)
    fmt.Println("[1:1]", len(sli4))
    fmt.Println("[1:1]", cap(sli4))
}
```

输出

```text
[1:4] [3 5 7]
[1:4] 3
[1:4] 5
-----
[1:3] [3 5]
[1:3] 2
[1:3] 5
-----
[1:2] [3]
[1:2] 1
[1:2] 5
-----
[1:1] []
[1:1] 0
[1:1] 5
```

当endIndex超出边界时候则会报错

```go
package main

import "fmt"

func main() {
    arr := []int{2, 3, 5, 7, 11, 13}
    sli1 := arr[1:0]
    fmt.Println("[1:0]", sli1)
    fmt.Println("[1:0]", len(sli1))
    fmt.Println("[1:0]", cap(sli1))
}
```

报错

```text
./slice.go:7: invalid slice index: 1 > 0
```
