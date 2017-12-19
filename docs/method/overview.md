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

### **有基类（也叫继承，使用接口）**

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

### **tar-gz**

python

```python
import tarfile
import os

def tar(fname):
    t = tarfile.open(fname + ".tar.gz", "w:gz")
    for root, dir, files in os.walk(fname):
        print root, dir, files
        for file in files:
            fullpath = os.path.join(root, file)
            t.add(fullpath)
    t.close()

if __name__ == "__main__":
    tar("del")
```

go

```go
package main

import (
    "fmt"
    "os"
    "io"
    "archive/tar"
    "compress/gzip"
)

func main() {
    // file write
    fw, err := os.Create("tar/lin_golang_src.tar.gz")
    if err != nil {
        panic(err)
    }
    defer fw.Close()

    // gzip write
    gw := gzip.NewWriter(fw)
    defer gw.Close()

    // tar write
    tw := tar.NewWriter(gw)
    defer tw.Close()

    // 打开文件夹
    dir, err := os.Open("file/")
    if err != nil {
        panic(nil)
    }
    defer dir.Close()

    // 读取文件列表
    fis, err := dir.Readdir(0)
    if err != nil {
        panic(err)
    }

    // 遍历文件列表
    for _, fi := range fis {
        // 逃过文件夹, 我这里就不递归了
        if fi.IsDir() {
            continue
        }

        // 打印文件名称
        fmt.Println(fi.Name())

        // 打开文件
        fr, err := os.Open(dir.Name() + "/" + fi.Name())
        if err != nil {
            panic(err)
        }
        defer fr.Close()

        // 信息头
        h := new(tar.Header)
        h.Name = fi.Name()
        h.Size = fi.Size()
        h.Mode = int64(fi.Mode())
        h.ModTime = fi.ModTime()

        // 写信息头
        err = tw.WriteHeader(h)
        if err != nil {
            panic(err)
        }

        // 写文件
        _, err = io.Copy(tw, fr)
        if err != nil {
            panic(err)
        }
    }
    fmt.Println("tar.gz ok")
}
```

总结核心代码：

python：

```python
// 创建tar.gz
t = tarfile.open(fname + ".tar.gz", "w:gz")
// 往tar里添加文件
t.add(fiepath)
```

go：

```go
// 创建tar.gz
fw, err := os.Create("tar/lin_golang_src.tar.gz")
gw := gzip.NewWriter(fw)
tw := tar.NewWriter(gw)
// 往tar里添加文件
fr, err := os.Open(filepath)
_, err = io.Copy(tw, fr)
```

可以看到：

1. python封装了做tar的所有细节（创建tar并gzip压缩），而golang没有封装细节，要自己依次创建一个文件，带给gzip，再将gzip的输出带给tar。

	1. 创建tar.gz：python只需要调用tarfile包自己的方法open即可，而golang分3步，先os.Create获得fw，再把fw带到gzip.NewWriter获得gw，再把gw带到tar.NewWriter获得tw

	2. 往tar里添加文件：python只需要调用tarfile包自己的方法add即可，而golang分2步，首先是通过os.Open（公共方法）获得文件句柄，再把文件句柄带到io.Copy（公共方法）来处理

2. 由于go语言的接口特点，golang没有必要像python一样完全封装细节，而是使用公共接口（每个类型都实现了相同的接口）

!!! note
	当然用golang自己可以再做一次封装，比如自己也搞个tarfile包，封装创建tar和gzip压缩方法

详细剖析上面的tar.gz例子，详见[golang-targz脑图](/attachment/golang_targz.itmz)（点击下载，并使用iThoughtsX打开）
