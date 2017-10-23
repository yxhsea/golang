slice是这样:

1. 在切片时候仅仅把原来元素的地址给拷贝过来了，即有当元素的值发生改变的时候，之前和之后引用的slice都可以看到变化

2. 一个slice中元素被删掉后不会影响其他slice。比如有3个slice都是来自同一个array或者slice，那么当这个slice的元素被删掉后，那3个slice和原来的array不受影响。应该是当那个元素被删掉后，go发现有被引用了，于是内存中并没有去删掉，因此那3个slice才不受影响，但是那3个slice的元素依然是来自原来的地址

还有一个特征也很重要，也表明了array和slice的区别：slice是地址，效果和指针相同，而array不是指针，举例说明：

```go
package main

import "fmt"

func arrDo(x [6]int) {
    x[2] = 333
}

func sliDo(x []int) {
    x[2] = 333
}

func main() {
    arr := [6]int{11, 22, 33, 44, 55, 66}
    sli := []int{1, 2, 3, 4, 5, 6}

    arrDo(arr)
    sliDo(sli)

    fmt.Println("arr:", arr)
    fmt.Println("sli:", sli)
}
```

输出

```text
arr: [11 22 33 44 55 66]
sli: [1 2 333 4 5 6]
```

!!! note ""
	从上面例子可以看出通过函数传递的array，在函数里把值修改后在函数外看不到。而slice是可以看到
