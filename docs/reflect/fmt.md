fmt在打印变量值的时候（例如fmt.Println或者fmt.Printf的%v）是调用reflect.Value来打印内部的值（如果是接口变量，则是hold的变量的值）。举例如下：

```text
fmt.Println(reflect.ValueOf(x))
```

这个例子中有2次调用reflect：

1. reflect.ValueOf(x)获得reflect.Value变量

2. Println调用reflect.ValueOf()

!!! note
	第二次调用reflect.ValueOf()是因为fmt对待reflect.Value和对待其他类型的变量是不一样的：

	- 若是reflect.Value则没有调用要打印变量的String()方法，而是打印其hold的准确值

	- 若非reflect.Value，则先查找变量是否有Error()，如果没有再查找String()
