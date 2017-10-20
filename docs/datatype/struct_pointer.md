```go
type Vertex struct {
	X int
	Y int
}

func main() {
	v := Vertex{1, 2}
	p := &v
	p.X = 1e9
	fmt.Println(v)
}
```

`p.X`原本应该写成`(*p).X`，但golang支持`p.X`这样写法，因此这2种写法的效果是一样的
