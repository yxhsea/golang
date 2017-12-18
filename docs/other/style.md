## **New风格**

---

??? note "例1：没有用New时候"
	person.go

	```go
	package person

	import "fmt"

	type Person struct {
		Name string
	}

	func (p *Person) SayHi() {
		fmt.Println("Hello, my name is", p.Name)
	}
	```

	调用者

	```go
	package main

	import (
		"github.com/cyent/repo1/person"
	)

	func main() {
		me := person.Person{Name:"Swaroop"}
		me.SayHi()
	}
	```

	输出

	```text
	Hello, my name is Swaroop
	```

	注意：导入的包里的struct的元素必须也要首字母大写，才能被调用者使用

??? note "例2：方法名带New可以让包看过去更像java的面向对象风格"
	person.go

	```
	package person

	import "fmt"

	type Person struct {
		name string
	}

	func NewPerson(n string) *Person {
		return &Person{name: n}
	}

	func (p *Person) SayHi() {
		fmt.Println("Hello, my name is", p.name)
	}
	```

	调用者

	```go
	package main

	import (
		"github.com/cyent/repo1/person"
	)

	func main() {
		me := person.NewPerson("Swaroop")
		me.SayHi()
	}
	```

	输出：

	```text
	Hello, my name is Swaroop
	```

	注意：在这个例子里用name而不需要首字母大写（Name）

什么时候用第一种例子（知道要调用包里的信息），什么时候用第二种例子（用New风格的函数包装），已知有2点原因来判断：

1. 需要封装的时候，使用New，不封装时候（比如一个模块内部，可以不用New）

	比如读取json文件，并转换为go的数据结构，会有2个事要做：

	1. 把json文件读入一个变量

	2. 把变量内容转换为go的数据结构

	这个时候如果就可以用第二种方法，就是函数包装，比如NewFromJsonFile()，这个方法把读文件、转换放在一起

2. 如果要传的参数比较少，可以不用New，如果要传的参数很多，有十几个，可以用New

!!! warning "struct和struct元素名需要大写才能被导入"
	详见[struct-导出名](/datatype/struct_export/)
