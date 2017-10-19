声明指针：

```go
var p *int
p = &i
```

??? note "其他声明方式"
	```go
	var p *int = &i
	```

	```go
	var p = &i
	```

	```go
	p := &i
	```

`&`符号会生成指针，如

```go
i := 42
p := &i
```

`*`符号读取指针指向的值，如

```go
fmt.Println(*p)
```

可以叠加，比如：

```go
i := 42
p := &i
q := &p
fmt.Printf("%T\n", p)
fmt.Printf("%T\n", q)
fmt.Println(*p)
fmt.Println(p)
fmt.Println(*q)
fmt.Println(**q)
```

输出

```text
*int
**int
42
0xc420056168
0xc420056168
42
```

也可以这么叠加，比如：

```go
x := 42
p := &x
*p = 11
*&*p = 12
*&*&*p = 13
fmt.Println(x)
```

输出

```text
13
```
