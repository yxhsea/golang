## **官网文档地址**

---

官网: https://blog.golang.org/laws-of-reflection

中文翻译: http://m.blog.csdn.net/xiaohu50/article/details/49496819

## **接口赋值的术语**

---

```go hl_lines="20"
package main

import "fmt"

type Adder interface {
	Add()
}

type MyStruct struct {
	X, Y int
}

func (this *MyStruct) Add() {
	fmt.Println(this.X + this.Y)
}

func main() {
	var i Adder
	s := &MyStruct{3, 4}
	i = s
	i.Add()
}
```

输出

```text
7
```

上面`i = s`可以用3个维度来阐述：

- assignment：赋值，将s赋值给i

- hold：存储的值，变量i hold s表示i存储的值是s

- item inside：和hold一样的意思（只不过主谓相反而已）比如item inside i中item表示s，因此可以理解为s inside i，即i包含了s

!!! note
	这3个维度的意思其实是一样的，在官方文档中可以看到交替出现

另外还有个单词implement：实现，由于\*MyStruct拥有方法Add()，因此\*MyStruct implement Adder接口（\*MyStruct实现了Adder接口）

## 学习反射前需要掌握的基础知识

---

### **go语言是静态类型**

切起：GO语言是静态类型语言，所有变量类型都是静态的：每个变量都有一个静态类型，每个变量的类型在编译时都是确定的，如int，float32, \*MyType, []byte等等。包括接口变量本身也是静态类型，接口变量的类型就是其接口（可以理解为接口名）。比如：

```text
var r io.Reader
r = os.Stdin
r = bufio.NewReader(r)
r = new(bytes.Buffer)
```

这里的r变量的类型始终是io.Reader

即接口变量的类型始终这个接口类型 ，而不是实现了这个接口变量的某type，无论这个接口变量hold哪个type

!!! warning
	目前还不找到办法查询到这个接口变量的类型，比如是io.Reader而不是io.Writer

### **基类型**

```text
type MyInt int
var i int
var j MyInt
```

这里i和j是不同的静态类型，尽管都拥有相同的underlying type（基类型），他们互相之间如果不进行转换是不能互相赋值的

### **go接口类型**

接口类型是十分重要的类型，接口变量可以存储任意的具体（除了接口）值，只要该值实现了接口的方法

有一个非常重要的接口类型叫做empty interface，即空接口：interface{}

有的人认为go接口是动态类型，这是错误的，go接口也是静态类型，go的类型都是静态

要始终记住go是静态类型语言

## **什么是反射**

---

反射用于检查自身的结构，属于元程序编程

每个语言的反射机制都是不同的，有的语言没有反射

## **go语言的反射**

---

### **go反射机制建立在类型系统之上**

### **接口类型的变量存储了一对值**

英文叫pair

> A variable of interface type stores a pair: the concrete value assigned to the variable, and that value's type descriptor. To be more precise, the value is the underlying concrete data item that implements the interface and the type describes the full type of that item.

```text
(value, type)
```

可以理解为(该接口变量hold的变量的值,该接口变量hold的变量的类型)

!!! warning "type是concrete type"
	接口变量逻辑上存储的值是(value, concrete type)而不是(value, interface type)，接口类型变量的值不能存储接口变量类型本身的类型。

!!! note "不用太在意pair"
	这个pair是底层的存储，是让人更加了解接口变量，实际使用的时候是不会看到的

例子

```text
var r io.Reader
tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0) //tty的类型是*os.File，而*os.File拥有Read()方法，io.Reader的方法集就是Read()
if err != nil {
    return nil, err
}
r = tty //tty实现了接口r，或者叫做tty赋值给了接口r
```

这里的(value, type)是(tty, \*os.File)
