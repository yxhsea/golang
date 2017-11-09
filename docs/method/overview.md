go没有类，但是在type上定义method，这样效果和类用起来很相似

如果说类是对数据和方法的抽象和封装，那么接口就是对类的抽象。

Golang 中没有 class 的概念，而是通过 interface 类型转换支持在动态类型语言中常见的鸭子类型从而达到运行时多态的效果。

关于面向对象、泛型、鸭子类型等概念详见[面向对象](http://127.0.0.1:8000/other/oo/)

## **网上搜到的interface资料**

---

- go中怎样形象的理解接口
https://segmentfault.com/q/1010000005140317

- 深入理解Go Interface：http://legendtkl.com/2017/06/12/understanding-golang-interface/

- 关于interface的理解: http://www.itnose.net/detail/6079953.html

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
