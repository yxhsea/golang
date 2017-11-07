Golang 中没有 class 的概念，而是通过 interface 类型转换支持在动态类型语言中常见的鸭子类型从而达到运行时多态的效果。

go没有类，但是在type上定义method，这样效果和类用起来很相似

## **比较python和go的面向对象写法**

---

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
