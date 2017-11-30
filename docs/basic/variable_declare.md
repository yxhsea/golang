## **var**

---

变量声明用`var`关键字，可以用在函数级别，也可以用在包级别，比如

```go
var c, python, java bool    //这个就是在包级别

func main() {
    var i int     //这个就是在函数级别
}
```

注意：

var多个变量时不能为每个变量设定类型（无论类型是否相同，无论是否有带初始值）比如：

| 语句 | 结果 |
| :-- | :-- |
| var i int, j int | 报错 |
| var i int, j int = 1, 2 | 报错 |
| var i string, j int | 报错 |
| var i string, j int = "a", 1 | 报错 |

var多个变量带初始值，如果类型不同，就不能指定变量类型，比如：

| 语句 | 结果 |
| :-- | :-- |
| var i, j = "aa", 2 | 正确 |
| var i, j int = "aa", 2 | 报错 |

var是否跟类型（包括var单个变量或者var多个变量）：

1. 跟类型：

	1. 带初始值: OK

		单个变量例子：`#!go var i int = 1`

		多个变量例子：`#!go var i, j int = 1, 2`

	2. 不带初始值: 根据类型自动设初始值，如int初始值为0，string初始值为""（空字符）

		单个变量例子：`#!go var i int`，则i为0

		多个变量例子：`#!go var i, j int`，则i和j为0

2. 不跟类型：

	1. 带初始值: 根据初始值自动判断类型，如1为int类型，"a"为string类型

		单个变量例子：`#!go var i = 1`，则i为int类型

		多个变量例子：

		1. `#!go var i, j = 1, 2`，则i和j为int类型

		2. `#!go var i, j = true, "a"`，则i为bool类型，j为string类型

	2. 不带初始值: 报错

		单个变量例子：`#!go var i`，会报错

		多个变量例子：`#!go var i, j`，会报错

也可以这么声明

```text
var (
	A int = 100
	B string
)
```

!!! warning "和const区别"
	var可以不赋值（会自动赋值初始值），而const必须赋值

## **:=**

---

`:=`是短声明变量，也叫简洁赋值语句

函数内，`:=` 用在明确类型的地方（但不能显式指定类型），可以用于替代`var`定义。

函数外（即包级别）的每个语句都必须以关键字开始（`var`、`func`等），`:=` 不能使用在函数外，如：

```go
var k = 3	//OK
k := 3		//报错

func main() {
	k := 3			//OK
	var k int = 3	//OK
	k int := 3		//报错
	var k := 3		//报错
	var k int := 3	//报错
}
```
