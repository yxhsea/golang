> A nil slice has a length and capacity of 0 and has no underlying array.

目前已知只有`var s []int`这1种方法才是nil slice。其他情况都不是，因为{==只有no underlying array才是nil slice==}

下面几种方法都不是nil slice

1. `s := []int{}`不是nil


2. 删除所有元素也不算是nil，如

	```go
	sli = sli[:cap(sli)]
	sli = sli[len(sli):]
	```

3. 用`make([]int, 0, 0)`也不是nil

实践证明上面的阐述

```go
package main

import "fmt"

func main() {
	var a []int
	fmt.Println("a:", a, len(a), cap(a))

	b := []int{}
	fmt.Println("b:", b, len(b), cap(b))

	c := make([]int, 0, 0)
	fmt.Println("c:", c, len(c), cap(c))

	d := []int{2, 3, 5, 7, 11, 13}
	d = d[:cap(d)]
	d = d[len(d):]
	fmt.Println("d:", d, len(d), cap(d))

	if a == nil {
		fmt.Println("a is nil")
	}

	if b == nil {
		fmt.Println("b is nil")
	}

	if c == nil {
		fmt.Println("c is nil")
	}

	if d == nil {
		fmt.Println("d is nil")
	}
}
```

输出

```text
a: [] 0 0
b: [] 0 0
c: [] 0 0
d: [] 0 0
a is nil
```
