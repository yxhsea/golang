**Reflection goes from reflection object to interface value.**

反射可以从反射类型对象到接口类型

我的理解，就是通过reflect.Value的Interface()方法反转得到最原始的变量，只不过得到的是一个接口类型的变量

!!! warning
	这个接口类型的变量是一个空接口，这里附上Interface()的定义

	```go
	// Interface returns v's current value as an interface{}.
	// It is equivalent to:
	//	var i interface{} = (v's underlying value)
	// It panics if the Value was obtained by accessing
	// unexported struct fields.
	func (v Value) Interface() (i interface{}) {
		return valueInterface(v, true)
	}
	```

[点击查看Interface()详情](/reflect/pkg_type_value/#interface)

简单来说，法则2和法则1是相反的，官方解释如下：

> Reflection goes from interface values to reflection objects and back again
