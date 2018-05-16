作用：缩小变量作用域，减少对全局变量的污染

## **先上例子**

---

```go
package main

import "fmt"

func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}
```

输出

```text
0 0
1 -2
3 -6
6 -12
10 -20
15 -30
21 -42
28 -56
36 -72
45 -90
```

!!! note
	```text
	pos, neg := adder(), adder()
	```

	相当于

	```text
	pos := adder()
	neg := adder()
	```

	var可以这么写

	```text
	var pos func(int) int
	var neg func(int) int
	pos = adder()
	reg = adder()
	```

上面的

```text
pos := adder()
```

可以理解为：执行adder()，然后将其return的内容赋值给pos，而return的内容就是一个标准的函数，即：

```go
func(x int) int {
	sum += x
	return sum
}
```

!!! note "因此，这个时候就是"
	```go
	pos := func(x int) int {
		sum += x
		return sum
	}
	```

## **闭包函数本身可用函数值书写**

---

比如上面这个例子，也可以写成：

```go
package main

import "fmt"

func main() {
    adder := func () func(int) int {
        sum := 0
        return func(x int) int {
            sum += x
            return sum
        }
    }

    pos, neg := adder(), adder()
    for i := 0; i < 10; i++ {
		fmt.Println(
		    pos(i),
		    neg(-2*i),
		)
    }
}
```

!!! note
	函数值内嵌函数值

输出也是

```text
0 0
1 -2
3 -6
6 -12
10 -20
15 -30
21 -42
28 -56
36 -72
45 -90
```

## **什么是闭包**

---

> 官方解释（译文）

> Go 函数可以是一个闭包。闭包是一个函数值，它引用了函数体之外的变量。 这个函数可以对这个引用的变量进行访问和赋值；换句话说这个函数被“绑定”在这个变量上。

例如，函数 adder 返回一个闭包。每个返回的闭包都被绑定到其各自的 sum 变量上。

在上面例子中（这里重新贴下代码，和上面代码一样）：

```go hl_lines="7 8 9 10"
package main

import "fmt"

func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}
```

上面背景高亮部分就是一个闭包，如pos := adder()的adder()表示返回了一个闭包，并赋值给了pos，同时，这个被赋值给了pos的闭包函数被绑定在sum变量上，因此pos闭包函数里的变量sum和neg变量里的sum毫无关系。

!!! note
	`func adder() func(int) int`的`func(int) int`表示adder()的输出值的类型是func(int) int这样一个函数

## **我对闭包的理解**

---

没有闭包的时候，函数就是一次性买卖，函数执行完毕后就无法再更改函数中变量的值（应该是内存释放了）；有了闭包后函数就成为了一个变量的值，只要变量没被释放，函数就会一直处于存活并独享的状态，因此可以后期更改函数中变量的值（因为这样就不会被go给回收内存了，会一直缓存在那里）。

比如，实现一个计算功能：一个数从0开始，每次加上自己的值和当前循环次数（当前第几次，循环从0开始，到9，共10次），然后*2，这样迭代10次：

没有闭包的时候这么写：

```go
func abc(x int) int {
    return x * 2
}

func main() {
    var a int
    for i := 0; i < 10; i ++ {
        a = abc(a+i)
        fmt.Println(a)
    }
}
```

如果用闭包可以这么写：

```go
func abc() func(int) int {
    res := 0
    return func(x int) int {
        res = (res + x) * 2
        return res
    }
}

func main() {
    a := abc()
    for i := 0; i < 10; i++ {
        fmt.Println(a(i))
    }
}
```

2种写法输出值都是：

```text
0
2
8
22
52
114
240
494
1004
2026
```

从上面例子可以看出闭包的3个好处：

1. 不是一次性消费，被引用声明后可以重复调用，同时变量又只限定在函数里，同时每次调用不是从初始值开始（函数里长期存储变量）

	!!! note ""
		这有点像使用面向对象的感觉，实例化一个类，这样这个类里的所有方法、属性都是为某个人私有独享的。但比面向对象更加的轻量化

2. 用了闭包后，主函数就变得简单了，把算法封装在一个函数里，使得主函数省略了a=abc(a+i)这种麻烦事了

3. 变量污染少，因为如果没用闭包，就会为了传递值到函数里，而在函数外部声明变量，但这样声明的变量又会被下面的其他函数或代码误改。

关于闭包的第一个好处，再啰嗦举个例子

1. 若不用闭包，则容易对函数外的变量误操作（误操作别人），例：

	```go
	var A int = 1
	func main() {
	    foo := func () {
	        A := 2
	        fmt.Println(A)
	    }
	    foo()
	    fmt.Println(A)
	}
	```

	输出：

	```
	2
	1
	```

	如果手误将A := 2写成了A = 2，那么输出就是：

	```text
	2
	2
	```

	即会影响外部变量A

2. 为了将某一个私有的值传递到某个函数里，就需要在函数外声明这个值，但是这样声明会导致这个值在其他函数里也可见了（别人误操作我），例：

	```go
	func main() {
	    A := 1
	    foo := func () int {
	        return A + 1
	    }
	    B := 1
	    bar := func () int {
	        return B + 2
	    }
	    fmt.Println(foo())
	    fmt.Println(bar())
	}
	```

	输出：

	```text
	2
	3
	```

	在bar里是可以对变量A做操作的，一个不小心就容易误修改变量A

	**结论：函数外的变量只能通过参数传递进去，不要通过全局变量的方式的渠道传递进去，当函数内能读取到的变量越多，出错概率(误操作)也就越高。**

## **最后举个例子**

---

实现斐波那契数列：

用闭包：

```go
func fibonacci() func() int {
    b1 := 1
    b2 := 0
    bc := 0
    return func() int {
        bc = b1 + b2
        b1 = b2
        b2 = bc
        return bc
    }
}

func main() {
    f := fibonacci()
    for i := 0; i < 10; i++ {
        fmt.Println(f())
    }
}
```

输出

```text
1
1
2
3
5
8
13
21
34
55
```

不用闭包：

```go
func fibonacci(num int) {
    b1 := 1
    b2 := 0
    bc := 0

    for i := 0; i < num; i ++ {
        bc = b1 + b2
        b1 = b2
        b2 = bc
        fmt.Println(bc)
    }
}

func main() {
    fibonacci(10)
}
```

这样输出也是正确的，但是这么写的话就将循环次数交给了fibonacci函数，即这个函数是个一次性使用的，当函数执行完毕后如果再执行函数，又是从初始值（这里是b1=1）开始，如果想能继续下去，就必须在函数外声明变量，但这样又造成了变量的泛滥（即对其他代码来说这几个变量是毫无意义，还可能造成对这几个变量的误操作），而有的时候想把for循环交由main控制，而是让fibonacci函数完成核心算法、核心数据存储，同时变量又不泛滥给其他代码

不用闭包也可以这么写：

```go
func main() {
    b1 := 1
    b2 := 0
    bc := 0
    fibonacci := func () {
        bc = b1 + b2
        b1 = b2
        b2 = bc
        fmt.Println(bc)
    }
    for i := 0; i < 10; i ++ {
        fibonacci()
    }
}
```

这样输出结果也是相同的，但是这么写的话，b1、b2、bc就变成了全局都能引用的变量了，而这3个变量其实只在fibonacci里用的到，所以这样把b1、b2、bc给放到全局就显得毫无意义，还有可能对这3个变量误操作
