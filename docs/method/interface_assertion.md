type assertions类型断言：

限制：**只能用于interface的值**

```text
t := i.(T)
```

断言interface i的值类型是T，如果是，t则为interface i的值，并且t的类型也为interface i的值类型；如果不是，则panic报错，类似：panic: interface conversion: interface {} is int, not float64

```text
t, ok := i.(T)
```

断言interface i的值类型是T，如果是，t则为interface i的值，并且t的类型也为interface i的值类型，ok为布尔类型的true；如果不是，不会报panic，而是ok为false，同时t的值类型为T，t的值为类型T的初始值（比如int、float64的初始值为0，string初始值为空字符串）

例子：

```go
package main

import "fmt"

func main() {
	var i interface{} = 100

	s, ok := i.(int)
	fmt.Println(s, ok)
	fmt.Printf("%T\n", s)

	f, ok := i.(float64)
	fmt.Println(f, ok)
	fmt.Printf("%T\n", f)
}
```

输出

```text
100 true
int
0 false
float64
```
