## **slice原理**

---

1. 在切片时候仅仅把原来元素的地址给拷贝过来了，即有当元素的值发生改变的时候，之前和之后引用的slice都可以看到变化

2. 一个slice中元素被删掉后不会影响其他slice。比如有3个slice都是来自同一个array或者slice，那么当这个slice的元素被删掉后，那3个slice和原来的array不受影响。应该是当那个元素被删掉后，go发现有被引用了，于是内存中并没有去删掉，因此那3个slice才不受影响，但是那3个slice的元素依然是来自原来的地址

## **关于literal**

---

1. 即使原来slice已经声明了，依然可以用literal赋值，但类型必须是和声明时候相同类型

2. 用literal赋值后（即使赋的值和原来的值一样），所有元素都是新的了，和原来没有关系了，如果原来有关联的slice，那么关联就解除了

```go
package main

import "fmt"

func main() {
	sli1 := []int{2, 3, 5, 7, 11, 13}
	sli2 := sli1[1:4]
	fmt.Println(sli1, len(sli1), cap(sli1))
	fmt.Println(sli2, len(sli2), cap(sli2))

	sli1 = []int{20, 30, 50, 70, 110, 130}
	sli2[2] = 500
	fmt.Println(sli1, len(sli1), cap(sli1))
	fmt.Println(sli2, len(sli2), cap(sli2))
}
```

输出

```text
[2 3 5 7 11 13] 6 6
[3 5 7] 3 5
[20 30 50 70 110 130] 6 6
[3 5 500] 3 5
```

## **关于关联脱离**

---

经测试，如下几种方法可以脱离关联

1. literal赋值，详见[](/datatype/arr_sli_key/#literal)

2. append()，详见[]()
