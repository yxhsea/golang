1. 如果一个类型实现了一个接口的所有方法，那么就说这个类型实现了这个接口

	```go
	type File interface {
	    Read(b Buffer) bool
	    Write(b Buffer) bool
	    Close()
	}

	type T struct {
	    ...
	}

	func (p T) Read(b Buffer) bool { return ... }
	func (p T) Write(b Buffer) bool { return ... }
	func (p T) Close() { ... }
	```

	上面例子可以这么描述：类型T实现了File接口的所有方法（即Read()、Write()、Close()），因此可以说类型T实现了接口File

2. 某类型实现某方法：比如上面这个例子，可以说类型T实现了方法Read()、Write()、Close()
