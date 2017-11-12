## **receiver可以使用指针**

---

```go
package main

import "fmt"

type MyStruct struct {
	x int
}

func (m MyStruct) Set1() {
	m.x = 1
}

func (m *MyStruct) Set2() {
	m.x = 2
}

func main() {
	s := MyStruct{x: 0}
	s.Set1()
	fmt.Println(s.x)
	s.Set2()
	fmt.Println(s.x)
}
```

输出

```text
0
2
```

可以看出，Set1并没有修改值，Set2才修改了值，是因为指针receiver才是修改原来的值，否则只是复制变量出来成为函数里的局部变量

## **function argument、method receiver、method argument关于pointer的区别**

---

function的argument和method的receiver都能接收pointer，区别：

1. function的argument类型必须和传入的类型一致，比如argument是pointer类型，则传入参数也必须是pointer，如果传入参数不是pointer类型，则传入参数不能是pointer。

2. method的receiver类型比较智能，如果receiver类型是pointer，那么当传入参数类型不是pointer时候，go会自动转为pointer。如果receiver类型不是pointer，那么当传入参数类型是pointer，会自动将pointer转换为pointer所对应的值。

	!!! warning
		接口的receiver类型必须和实现接口一致，否则会报错，详见[接口章节部分](/method/interface_main/#receiver)

	如果将上面的例子稍微修改下

	```go hl_lines="18"
	package main

	import "fmt"

	type MyStruct struct {
		x int
	}

	func (m MyStruct) Set1() {
		m.x = 1
	}

	func (m *MyStruct) Set2() {
		m.x = 2
	}

	func main() {
		s := &MyStruct{x: 0}
		s.Set1()
		fmt.Println(s.x)
		s.Set2()
		fmt.Println(s.x)
	}
	```

	输出不变，也是

	```text
	0
	2
	```

	上面的`s.Set1()`会被go自动转为`(*s).Set1()`

	同理，如果改为`s := MyStruct{x: 0}`，那么`s.Set2()`会被go自动转为`(&s).Set2()`

method argument和function argument是相同的，类型必须一致

!!! note
	receiver在其他语言以及go语言里也叫做 **函数签名**（函数签名是最普遍的叫法）

## **如何判断receiver是否要用指针**

---

method的value receiver和pointer receiver怎么选择，官方说了2个原因来使用pointer receiver

1. 需要修改原来的值

2. 防止每次调用method时候都拷贝value，比如当struct很大的时候每次都拷贝value会降低效率

下面是我个人理解，不一定准确：

实际应用中，有一个情况是不能用pointer receiver：pointer可能造成安全风险，例如某个method只是要获取金额，而不是修改金额，那么使用pointer的话就存在当内存被泄露等情况导致金额被修改。

因此，若没有涉及性能问题，且method的功能是读，而非写的时候，首选value receiver
