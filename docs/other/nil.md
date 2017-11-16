## **int和[]int nil**

---

```go linenums="1"
package main

import "fmt"

func main() {
	var a int
	if a == nil {
		fmt.Println("a is nil")
	}
}
```

输出

```text
./nil.go:7: cannot convert nil to type int
```

修改代码

```go linenums="1"
package main

import "fmt"

func main() {
	var a int = nil
	if a == nil {
		fmt.Println("a is nil")
	}
}
```

输出

```text
./nil.go:6: cannot use nil as type int in assignment
./nil.go:7: cannot convert nil to type int
```

修改代码

```go linenums="1"
package main

import "fmt"

func main() {
	var a []int
	if a == nil {
		fmt.Println("a is nil")
	}
}
```

输出

```text
a is nil
```

修改代码

```go linenums="1"
package main

import "fmt"

func main() {
	var a []int = nil
	if a == nil {
		fmt.Println("a is nil")
	}
}
```

输出

```text
a is nil
```

**说明不是什么所有类型的变量都能用if xxx == nil来判断是否是nil**

## **array和slice nil**

---

[详见array slice章节](/datatype/arr_sli_nil/)

## **map nil**

---

[详见map章节](/datatype/map_nil/)

## **interface nil**

---

[详见接口章节](/method/interface_nil/)
