## **copy使用**

---

```go
a := []int{2, 3, 5, 7, 11, 13}
d := make([]int, len(a))
copy(d, a)
```

上面将a的元素复制给了d，是复制而不是引用

1. append和copy都是可以将元素内容拷贝给另一个slice（用于复制而不是引用），这样新生成的slice就和原来的slice彻底脱离关系了（复制元素内容到新的内存地址）

2. append当容量不够时候会扩大cap，而扩大cap是将元素内容复制出来，因此从原来slice衍生出来的slice都将和这个slice脱离关系

3. copy无法自己复制给自己，即`copy(a,a)`这样虽然不报错，但是没有任何效果（没有和衍生的slice脱离关系）

4. copy里第2个参数也必须是slice，且类型要和第一个参数一致

## **当复制的元素和被复制的元素个数不一致时**

---

1. 不会扩大被复制的cap

2. 会在len范围内产生覆盖

```go
package main

import "fmt"

func main() {
	a := []int{1, 2, 3, 4, 5}

	s1 := make([]int, 5, 10)
	fmt.Println(s1)
	copy(s1, a)
	fmt.Println(s1, len(s1), cap(s1))

	fmt.Println("-----")

	s2 := make([]int, 3, 10)
	fmt.Println(s2)
	copy(s2, a)
	fmt.Println(s2, len(s2), cap(s2))

	fmt.Println("-----")

	s3 := make([]int, 7, 10)
	fmt.Println(s3)
	copy(s3, a)
	fmt.Println(s3, len(s3), cap(s3))

	fmt.Println("-----")

	s4 := make([]int, 3, 3)
	fmt.Println(s4)
	copy(s4, a)
	fmt.Println(s4, len(s4), cap(s4))
}
```

输出

```text
[0 0 0 0 0]
[1 2 3 4 5] 5 10
-----
[0 0 0]
[1 2 3] 3 10
-----
[0 0 0 0 0 0 0]
[1 2 3 4 5 0 0] 7 10
-----
[0 0 0]
[1 2 3] 3 3
```
