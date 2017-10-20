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
