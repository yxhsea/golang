```go
type Key struct {
    a, b string
}

type Value struct {
    c, d int
}

var m map[Key]Value

func main() {
    m = make(map[Key]Value)
    m[Key{
        "A", "B",
    }] = Value{
        1, 2,
    }
    fmt.Println(m[Key{
        "A","B",
    }])
    fmt.Println(m)
    fmt.Printf("%T\n", m)
}
```

输出

```text
{1 2}
map[{A B}:{1 2}]
map[main.Key]main.Value
```

!!! note
	1. Key、Value写在函数外和写在main函数里，都是main.key、main.Value

	2. 上面的Key、Value、m的声明都可以放到main函数里写
