empty interface空接口

前面学习的变量都要在声明时候确定类型，虽然用:=可以不用指定类型，但实际是类型推倒，在赋值后类型就确定下来不能改了。

空接口的作用：不用指定类型变量，并且类型可变。比如：

```text
var i interface{}
i = 42	//这个时候i就是int类型
i = "hello"	//这个时候i就是string类型
```

!!! note
	interface{}这么写比较方便，当然也写成2行：

	```text
	type I interface {
	}
	```

完整例子

```go
package main

import "fmt"

type I interface{}

func main() {
	var i I
	i = 42 //这个时候i就是int类型
	fmt.Printf("%v,%T\n", i, i)
	i = "hello" //这个时候i就是string类型
	fmt.Printf("%v,%T\n", i, i)
}
```

输出

```text
42,int
hello,string
```
