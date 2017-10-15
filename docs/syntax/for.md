## **基本格式**

---

golang里循环只有for一种，没有其他的（比如其他语言还有while）

语法：

```go
for init statement; condition expression; post statement {
    ...
}
```

!!! note "没有括号"
	`init statement; condition expression; post statement`前后没有括号`()`

举例

```go
for i := 0; i < 10; i++ {
    sum += i
}
```

!!! warning "init statement里不能用var"
	这里`i := 0`不能写成`var i int = 0`，因为这个地方要求不能用`var`，只能用`:=`，如果一定要`var`，要在`for`前先`var`，比如这样也是可以的：

	```go
	var i int
	for i = 0; i < 10; i ++ {
	    sum += i
	}
	```

## **简写格式**

---

init statement和post statement是可选的，因此可以忽略，比如：

```go
sum := 1
for ; sum < 1000; {
    sum += sum
}
```

可以省略分号，即可以写成（这种写法相当于C语言中的while）

```go
sum := 1
for sum < 1000 {
    sum += sum
}
```

!!! warning "分号只能都要，或者都不要"
	下面这2种是错的

	```go
	for ; sum < 1000 {
	    sum += sum
	}
	```

	或

	```go
	for sum < 1000; {
	    sum += sum
	}
	```

## **死循环**

---

```go
for {
}
```

## **结合range**

---

结合range可以对slice、map、数组、字符串等进行迭代循环

```go
for key, value := range oldMap {
    newMap[key] = value
}
```

比如：

```go
func main() {
    numbers := [6]int{1, 2, 3, 5}

    for i, x := range numbers {
        fmt.Printf("第 %d 位 x 的值 = %d\n", i, x)
    }
}
```
