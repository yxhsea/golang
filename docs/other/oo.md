偶然在一个QQ技术群里看到的一个图片，挺生动的:

![](/img/oo.jpg)

## **面向对象3步**

---

详见：http://developer.51cto.com/art/200906/130568.htm

1. 系统分析

2. 系统设计

3. 实现（编码）

## **重载overload与覆盖override**

---

http://developer.51cto.com/art/201106/266705.htm

Overload是重载的意思，Override是覆盖的意思，也就是重写。

重载Overload表示同一个类中可以有多个名称相同的方法，但这些方法的参数列表各不相同（即参数个数或类型不同）。

重写Override表示子类中的方法可以与父类中的某个方法的名称和参数完全相同，通过子类创建的实例对象调用这个方法时，将调用子类中的定义方法，这相当于把父类中定义的那个完全相同的方法给覆盖了，这也是面向对象编程的多态性的一种表现。

**重写override是父类与子类之间多态性的一种表现，重载overload是一个类中多态性的一种表现**，go语言只支持override，不支持overload

??? note "go不支持overload的例子"
	```go
	package main

	import "fmt"

	type MyStruct struct {
		X, Y int
	}

	func (this *MyStruct) foo(i int) {
		fmt.Println(this.X + i)
	}

	func (this *MyStruct) foo(j string) {
		fmt.Println(this.X)
		fmt.Println(j)
	}

	func main() {
		s := MyStruct{3,4}
		s.foo(100)
		s.foo("hello")
	}
	```

	报错

	```text
	./t.go:13:6: (*MyStruct).foo redeclared in this block
		previous declaration at ./t.go:9:6
	./t.go:13:6: method redeclared: MyStruct.foo
		method(*MyStruct) func(int)
		method(*MyStruct) func(string)
	./t.go:21:8: cannot use "hello" (type string) as type int in argument to s.foo
	```

## **多态**

---

http://bbs.itheima.com/thread-119448-1-1.html

不要把函数重载理解为多态。

多态是一种运行期的行为，不是编译期的行为。

多态可以理解为 一个事物，多种表现形态。

比如一个老师（好比java中一个类，一个描述老师的类），在学生面前他以老师的形态存在，而在他孩子面前以父亲的形式存在，在他妻子面前则以老公的形式存在。

例如在java中可以用父类类型的引用指向子类类型

```text
Object obj=new String("a");
```

obj为父类类型引用，用obj指向了子类对象String。

不用多态时，这样写

```text
String s=new String("a");
```

即用本类类型引用指向本类类型实例。

!!! note "关于java中的多态"
	下面2个都是多态：

	1. 父类的引用类型变量指向了子类的对象

	2. 接口类型的引用类型变量指向了接口实现类 的对象。

	以下是第2点的例子：

	实现关系下的多态：
	接口 变量 = new 接口实现类的对象。

	```java
	//接口的方法全部都是非静态的方法
	interface Dao{
	    public void add();  
	    public void delete();  
	}  

	//接口的实现类  
	class UserDao implements Dao{  
	    public void add(){  
	        System.out.println("添加员工成功！！");  
	    }   
	    public void delete(){  
	        System.out.println("删除员工成功！！");  
	    }  
	}  

	class Demo3   
	{  
	    public static void main(String[] args)   
	    {  
	        //实现关系下的多态  
	        Dao d = new UserDao(); //接口的引用类型变量指向了接口实现类的对象。  
	        d.add();  
	    }  
	}
	```

## **鸭子类型**

---

### **介绍**

https://zh.wikipedia.org/wiki/鸭子类型

http://www.jb51.net/article/116025.htm（鸭子类型与多态）

当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子

在程序设计中，鸭子类型（英语：Duck typing）是动态类型和某些静态语言的一种对象推断风格。

在鸭子类型中，关注的不是对象的类型本身，而是它是如何使用的。

鸭子类型通常得益于不测试方法和函数中参数的类型，而是依赖文档、清晰的代码和测试来确保正确使用。

个人理解鸭子类型是多态的一种实现方式

### **python鸭子类型**

https://docs.python.org/3/glossary.html#term-duck-typing

Python不支持多态，也不用支持多态，python是一种多态语言，崇尚鸭子类型

例1

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

def calculate(a, b, c):
	return (a+b)*c

example1 = calculate(1, 2, 3)
example2 = calculate([1, 2, 3], [4, 5, 6], 2)
example3 = calculate('apples ', 'and oranges, ', 3)

print example1
print example2
print example3
```

输出

```text
9
[1, 2, 3, 4, 5, 6, 1, 2, 3, 4, 5, 6]
apples and oranges, apples and oranges, apples and oranges,
```

!!! note ""
	该例子鸭子类型在不使用继承的情况下使用了多态

例2

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

class Duck:
    def quack(self):
        print "这鸭子在呱呱叫"
    def feathers(self):
        print "这鸭子拥有白色与灰色羽毛"

class Person:
    def quack(self):
        print "这人正在模仿鸭子"
    def feathers(self):
        print "这人在地上拿起1根羽毛然后給其他人看"

def in_the_forest(duck):
    duck.quack()
    duck.feathers()

def game():
    donald = Duck()
    john = Person()
    in_the_forest(donald)
    in_the_forest(john)

game()
```

输出

```text
这鸭子在呱呱叫
这鸭子拥有白色与灰色羽毛
这人正在模仿鸭子
这人在地上拿起1根羽毛然后給其他人看
```

### **go鸭子类型**

golang中没有class的概念，而是通过interface类型转换支持动态类型语言中常见的鸭子类型，来达到运行时多态的效果。

接口可以提供鸭子类型的一些益处，但鸭子类型与之不同的是没有显式定义任何接口。

上面python的例子用go可以这么写：

```go
package main

import "fmt"

type Ducker interface {
	quack()
	feathers()
}

type Duck struct{}
type Person struct{}

func (this *Duck) quack() {
	fmt.Println("这鸭子在呱呱叫")
}

func (this *Duck) feathers() {
	fmt.Println("这鸭子拥有白色与灰色羽毛")
}

func (this *Person) quack() {
	fmt.Println("这人正在模仿鸭子")
}

func (this *Person) feathers() {
	fmt.Println("这人在地上拿起1根羽毛然后給其他人看")
}

func in_the_forest(d Ducker) {
	d.quack()
	d.feathers()
}

func main() {
	donald := &Duck{}
	john := &Person{}
	in_the_forest(donald)
	in_the_forest(john)
}
```

输出也是

```text
这鸭子在呱呱叫
这鸭子拥有白色与灰色羽毛
这人正在模仿鸭子
这人在地上拿起1根羽毛然后給其他人看
```

这2个例子对比，可以直接看出几点区别:

1. python无需接口（python是动态语言，没有接口），而go需要接口

2. python的in_the_forest无需指定传入参数类型，而go需要（传入参数时候实现了接口）

## **泛型编程**

---

[go"泛型编程"](http://legendtkl.com/2015/11/25/go-generic-programming/)

## **抽象类与接口**

---

抽象类（也叫基类）：

- 小明在家跟妈妈说，我要吃水果。妈妈去水果摊去买水果，看看哪个便宜就买哪个。最后买回去了橘子。这就叫抽象类。

- 你有一辆宝马的汽车，有一天，你拿它去越野了，轮胎，哎，自然爆掉，现在你想换轮胎。你不想换宝马的轮胎，太贵。你不想换qq的轮胎，不好使。你想换奥迪的，换了，好用。这就是抽象类。

接口：

- 你在街上被人群殴，有人用棍子打你，有人用脚踹，有人用拳头。反正你费血了。这就叫接口。

- 你的电脑上只有一个USB接口。这个USB接口可以接MP3，数码相机，摄像头，鼠标，键盘等。所有的上述硬件都可以公用这个接口，而且有很好的扩展性。这就是接口

接口和抽象类都是多态。

假设我要拿遥控器打开电器。接口和抽象类多态的侧重点不同。接口的侧重点在遥控器，多态的侧重点在电器

横看接口竖看类：横向扩展使用接口；纵向扩展使用抽象基类；横向和纵向都要扩展，使用抽象基类继承接口。

- 在差异较大的对象中追求功能上的共性时，使用接口。

- 在差异较小的对象中追求功能上的不同时，使用抽象基类，因为抽象基类可以包含实现的成员。

## **golang里的"面向对象"**

---

[详见接口章节](/method/interface_oo/)

推荐小白看看: [从无到有逐步理解面向对象和接口](/method/interface_01/)

## **3种代码风格**

---

来自：https://docs.python.org/3/glossary.html 中提到的

1. duck typing

2. EAFP（Easier to ask for forgiveness than permission）

3. LBYL（Look before you leap）

## **泛型编程、鸭子类型、interface关系**

---

这段话为个人理解，不保证准确性鸭子类型是一个老外提出的构想，是一种动态的代码风格，某个开发语言支持了这个构想而产生的编程风格或规范，叫做泛型编程，interface是go对泛型编程的实现（准确说是go不支持泛型编程，但go的interface可以实现类似泛型编程的效果）

泛型编程：像类一样，泛型也是一种抽象数据类型，但是泛型不属于面向对象，它是面向对象的补充和发展。

## **继承**

---

使用struct嵌套可以模拟继承，详见[面向对象写法比较-继承](/method/overview/#_2)
