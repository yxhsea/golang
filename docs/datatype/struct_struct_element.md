go可以模拟面向对象中的继承（准确叫法是嵌入，英语叫embed，而不是inherit）

```go
package main

import "fmt"

type Dog struct {
	name string
}

type BDog struct {
	Dog
	name string
}

func (this *Dog) callMyName() {
	fmt.Printf("my name is %q\n", this.name)
}

func main() {
	b := new(BDog)
	b.name = "this is a BDog name"
	b.Dog.name = "this is a Dog name"
	b.callMyName()
}
```

!!! note
	上面`b := new(BDog)`可以用`b := BDog{}`替代，但这2者是有一点区别的，区别在于前者是指针，详见[new](/other/new/#new)

输出

```text
my name is "this is a Dog name"
```

!!! warning
	上面callMyName是Dog才有的方法，而BDog没有，但是却能输出，并且输出的是Dog.name的内容。

	因为：Go编译器在编译检查过程（不是在运行时候检查）中，会去遍历结构中的成员，并检查成员中是否有支持callMyName()的调用。如果只找到一个成员支持这种调用，那么b.callMyName()是合法的。如果是两个或者两个以上的成员都支持，那么就会报一个调用有歧义的错误"ambiguous selector b.callMyName"。

扩展一下，把Dog放到一个包里，导入进来：

dog.go

```go hl_lines="6"
package dog

import "fmt"

type Dog struct {
	Name string
}

func (this *Dog) CallMyName() {
	fmt.Printf("Dog my name is %q\n", this.Name)
}
```

!!! note ""
	注意Dog struct里Name改成了大写

main.go

```go hl_lines="12"
package main

import "github.com/cyent/repo1/dog"

type BDog struct {
	dog.Dog
	name string
}

func main() {
	b := new(BDog)
	b.Name = "this is a Dog name"
	b.CallMyName()
}
```

!!! note ""
	注意b.Name是大写

输出也是

```text
Dog my name is "this is a Dog name"
```

可以把b.Name改为b.Dog.Name，即

```go hl_lines="12"
package main

import "github.com/cyent/repo1/dog"

type BDog struct {
	dog.Dog
	name string
}

func main() {
	b := new(BDog)
	b.Dog.Name = "this is a Dog name"
	b.CallMyName()
}
```

输出也是

```text
Dog my name is "this is a Dog name"
```

b.Name和b.Dog.Name是有区别的，只要把BDog结构体里的name也改成Name就看到区别了

```go hl_lines="7"
package main

import "github.com/cyent/repo1/dog"

type BDog struct {
	dog.Dog
	Name string
}

func main() {
	b := new(BDog)
	b.Dog.Name = "this is a Dog name"
	b.CallMyName()
}
```

输出

```text
Dog my name is "this is a Dog name"
```

现在改成

```go hl_lines="12"
package main

import "github.com/cyent/repo1/dog"

type BDog struct {
	dog.Dog
	Name string
}

func main() {
	b := new(BDog)
	b.Name = "this is a Dog name"
	b.CallMyName()
}
```

输出

```text
Dog my name is ""
```

可以看到区别：当自身struct里的元素名和embed导入的元素名重复时候，b.Name是会赋值给自身的元素，此时只有b.Dog.Name才能显式指定导入的元素

所以：如果要给导入的元素赋值，应该要用b.Dog.Name这种方式显式赋值
