## **基本格式**

---

```go
if x > 0 {
    ...
}
```

## **短声明**

---

```go
if v := math.Pow(x, n); v < lim {
    ...
}
```

!!! warning ""
	v是局部变量，只在if语句里有效（包括else）

当然也可以
var a int
if a = 1; a < 2 {
    fmt.Println("yes")
}

if else写法：
if ... {
    ...
} else if ... {
} else {
    ...
}
