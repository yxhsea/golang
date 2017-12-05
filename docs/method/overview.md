go没有类，但是在type上定义method，这样效果和类用起来很相似

如果说类是对数据和方法的抽象和封装，那么接口就是对类的抽象。

Golang 中没有 class 的概念，而是通过 interface 类型转换支持在动态类型语言中常见的鸭子类型从而达到运行时多态的效果。

关于面向对象、泛型、鸭子类型等概念详见[面向对象](/other/oo/)

## **网上搜到的interface资料**

---

- go中怎样形象的理解接口
https://segmentfault.com/q/1010000005140317

- 深入理解Go Interface：http://legendtkl.com/2017/06/12/understanding-golang-interface/

- 关于interface的理解: http://www.itnose.net/detail/6079953.html

## **比较python和go的面向对象写法**

---

### **无基类（无接口）**

python

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

class Person:
	def __init__(self, name):
		self.name = name

	def printName(self):
		print self.name

me = Person('cyent')
me.printName()
```

go

```go
package main

import "fmt"

type Person struct {
	name string
}

func (p *Person) printName() {
	fmt.Println(p.name)
}

func main() {
	me := Person{name: "cyent"}
	me.printName()
}
```

输出都是

```text
cyent
```

### **有基类（使用接口）**

python

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

class SchoolMember:
	def __init__(self, name, age):
		self.name = name
		self.age = age

	def tell(self):
		pass

class Teacher(SchoolMember):
	def __init__(self, name, age, salary):
		SchoolMember.__init__(self, name, age)
		self.salary = salary

	def tell(self):
		SchoolMember.tell(self)
		print 'Salary: "%d"' % self.salary

class Student(SchoolMember):
	def __init__(self, name, age, marks):
		SchoolMember.__init__(self, name, age)
		self.marks = marks

	def tell(self):
		SchoolMember.tell(self)
		print 'Marks: "%d"' % self.marks

t = Teacher('Mrs. Shrividya', 40, 30000)
s = Student('Swaroop', 22, 75)

t.tell()
s.tell()
```

输出

```text
Salary: "30000"
Marks: "75"
```

go

```go
package main

import "fmt"

type SchoolMember interface {
	tell()
}

type Teacher struct {
	name string
	age int
	salary int
}

type Student struct {
	name string
	age int
	marks int
}

func (this *Teacher) tell() {
	fmt.Printf("Salary: %d\n", this.salary)
}

func (this *Student) tell() {
	fmt.Printf("Marks: %d\n", this.marks)
}

func main() {
	var t SchoolMember = &Teacher{"Mrs. Shrividya", 40, 30000}
	var s SchoolMember = &Student{"Swaroop", 22, 75}
	t.tell()
	s.tell()
}
```

输出

```text
Salary: 30000
Marks: 75
```
