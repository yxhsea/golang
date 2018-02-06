```go
type vertex struct {
	x int
	y int
}

func main() {
	fmt.Println(vertex{1, 2})	//打印整个struct
	v := vertex{1, 2}
	v.x = 4
	fmt.Println(v.x)			//仅打印x这个字段
	fmt.Printf("%T\n", v)
}
```

输出

```text
{1 2}
4
main.vertex
```

上面这种方法最常见，但要注意，v这个变量的类型是vertex（准确说是main.vertex），而不是struct。因为真正的struct类型应该是以下例子这样直接用struct{}声明

```go
package main

import "fmt"

type vertex struct {
	a, b int
	c    string
}

func main() {
	s1 := struct{a, b int; c string}{1, 2, "hello"}
	fmt.Printf("%T,%v,%+v\n", s1, s1, s1)

	s2 := struct{a, b int; c string}{a:1, b:2, c:"hello"}
	fmt.Printf("%T,%v,%+v\n", s2, s2, s2)

	s3 := struct{a int; b int; c string}{1, 2, "hello"}
	fmt.Printf("%T,%v,%+v\n", s3, s3, s3)

	s4 := struct{a int; b int; c string}{a:1, b:2, c:"hello"}
	fmt.Printf("%T,%v,%+v\n", s4, s4, s4)

	s5 := vertex{a:1, b:2, c:"hello"}
	fmt.Printf("%T,%v,%+v\n", s5, s5, s5)
}
```

输出

```text
struct { a int; b int; c string },{1 2 hello},{a:1 b:2 c:hello}
struct { a int; b int; c string },{1 2 hello},{a:1 b:2 c:hello}
struct { a int; b int; c string },{1 2 hello},{a:1 b:2 c:hello}
struct { a int; b int; c string },{1 2 hello},{a:1 b:2 c:hello}
main.vertex,{1 2 hello},{a:1 b:2 c:hello}
```

可以看到直接用struct{}声明的变量的类型才是struct{...}

另外，空struct这么表示:

```text
struct{}{}
```
