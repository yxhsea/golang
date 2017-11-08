首字母大写才可以被其他模块导入引用，包含struct名和其中元素，只有大写的才可被其他模块导入引用。例如

1. 这样都不能被其他模块引用

	```go
	type vertex struct {
		x int
		y int
	}
	```

2. 这样只有struct可以被其他模块引用，但是struct中的元素不能被其他模块引用

	```go
	type Vertex struct {
		x int
		y int
	}
	```

3. 这样struct和struct中的元素都可以被其他模块引用

	```go
	type Vertex struct {
		X int
		Y int
	}
	```
