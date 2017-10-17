## **跟变量**

---

某个变量等于A或者等于B

```go
switch varA {
case 0:
    ...
case 1:
    ...
default:
    ...
}
```

## **不跟变量**

---

相当于if-else if-else if-else

```go
switch {
case varA > 2:
    ...
case varA > 0:
    ...
default:
    ...
}
```

`switch { ... }` 等同于 `switch true { ... }`

## **短声明**

---

2种短声明形式：

1. 跟变量

	```go
	switch os := runtime.GOOS; os {
	case "darwin":
	    fmt.Println("OS X.")
	case "linux":
	    fmt.Println("Linux.")
	default:
	    fmt.Printf("%s.", os)
	}
	```

	!!! note ""
		os变量作用域只在switch里

2. 不跟变量：

	```go
	switch varA := 5; {	//注意这里有个分号，如果没有分号会报错
	case varA < 5:
	    fmt.Println("<5")
	case varA == 5:
	    fmt.Println("==5")
	case varA > 5:
	    fmt.Println(">5")
	}
	```

就是说switch分号后面如果是一个变量，则case就是判断这个变量值等于多少，如果分号后面没变量，则case写条件句，当true时触发

**注意: case里不能用短声明或赋值语句**

## **作用域**

---

??? note "例1. switch的短声明是独立的作用域吗?"
	```go
	package main

	import "fmt"

	func main() {
		a := 3

		switch a := 5; { //注意这里有个分号，如果没有分号会报错
		case a < 5:
			fmt.Println("<5")
		case a == 5:
			fmt.Println("==5")
		case a > 5:
			fmt.Println(">5")
		}

		fmt.Println(a)
	}
	```

	输出

	```text
	==5
	3
	```

	答: switch的短声明是独立的作用域

??? note "例2. switch的每个case均是独立的作用域吗?"
	```go
	package main

	import "fmt"

	func main() {
		i := 1

		switch a := 5; { //注意这里有个分号，如果没有分号会报错
		case a < 5:
			fmt.Println("<5")
			i := 2
			fmt.Println(i)
		case a == 5:
			fmt.Println("==5")
			i := 3
			fmt.Println(i)
		case a > 5:
			fmt.Println(">5")
			i := 4
			fmt.Println(i)
		}

		fmt.Println(i)
	}
	```

	输出

	```text
	==5
	3
	1
	```

	答: switch的每个case均是独立的作用域

??? note "例3. switch使用赋值语句时是独立变量还是引用外部变量?"
	```go
	package main

	import "fmt"

	func main() {
		a := 3

		switch a = 5; { //注意这里有个分号，如果没有分号会报错
		case a < 5:
			fmt.Println("<5")
		case a == 5:
			fmt.Println("==5")
		case a > 5:
			fmt.Println(">5")
		}

		fmt.Println(a)
	}
	```

	输出

	```text
	==5
	5
	```

	答: switch使用赋值语句时是引用外部变量
