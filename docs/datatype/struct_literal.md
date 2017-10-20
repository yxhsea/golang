```go
type Vertex struct {
	X, Y int
}

var (
	v1 = Vertex{1, 2}  // has type Vertex
	v2 = Vertex{X: 1}  // Y:0 is implicit
	v3 = Vertex{}      // X:0 and Y:0
	p  = &Vertex{1, 2} // has type *Vertex
)

func main() {
	fmt.Println(v1, p, v2, v3)
	fmt.Printf("%T\n", p)
	fmt.Printf("%T\n", *p)
}
```

输出

```text
{1 2} &{1 2} {1 0} {0 0}
*main.Vertex
main.Vertex
```

!!! note "Note"
	1. 声明多个变量时候可以用`var (...)`
	2. `v2 = Vertex{X: 1}`这样表示X为1，其他都为类型推导的默认值
	3. 这里struct指针类型是`*main.Vertex`
