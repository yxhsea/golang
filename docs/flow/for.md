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

!!! note
	使用range的话就不能在前后使用分号了

[详见range详解](/other/range/)

## **作用域**

---

以下均为个人理解，不保证准确性

1. for的init statement里声明的变量，以及代码块里声明的变量，都仅限于for里，在外部是无法引用的

	??? note "例子"
		```go
		package main

		import "fmt"

		func main() {
			for i := 0; i < 2; i ++ {
				fmt.Println(i)
				a := 100
			}
			fmt.Println(i)
			fmt.Println(a)
		}
		```

		报错

		```text
		./for.go:10: undefined: i
		./for.go:11: undefined: a
		```

2. 每次循环都是一个新的空间，即在第一次循环时候声明的变量，在下一次循环时候是看不到的

	??? note "例子"
		```go
		package main

		import "fmt"

		func main() {
			var a int = 100
			fmt.Println(a)

			fmt.Println("=====")
			for i := 0; i < 2; i ++ {
				fmt.Println("---")
				a += 1
				fmt.Println(a)
				a := 200
				fmt.Println(a)
				a += 1
				fmt.Println(a)
			}
			fmt.Println("=====")

			fmt.Println(a)
		}
		```

		输出

		```text
		100
		=====
		---
		101
		200
		201
		---
		102
		200
		201
		=====
		102
		```

		可以看到每次循环时候a都是200，而不会传递到下次循环中
