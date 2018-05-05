用T(v)表达式来将变量v转换成类型T

!!! warning "必须指定类型才能转换"
	var a int = 5，那么：

	var b string = string(a) 是OK的，而var b string = a是会报错的

另外，var b = string(a) 也可以这么写 b := string(a)

**注意：若类型T是个指针，则需要用圆括号，比如`(*pointer)(r)`**，详见[指针章节](/datatype/pointer/#_3)

!!! note ""
	类型转换和类型断言的区别详见[接口章节-深入理解](/method/interface_deep/#_6)
