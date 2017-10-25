slice可以嵌套包含slice，并且可以嵌套多层

嵌套一层

```go
board1 := [][]string{
	[]string{"_", "_", "_"},
	[]string{"_", "_", "_"},
	[]string{"_", "_", "_"},
}
```

嵌套两层

```go
board2 := [][][]string{
	[][]string{
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
	},
	[][]string{
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
	},
	[][]string{
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
	},
}
```

赋值

```go
board1[0][0] = "X"
board2[0][0][0] = "Y"
```

通过嵌套实现了高级数据结构，比如python中列表嵌套字典嵌套列表等等

!!! note "分解说明"
	`board1 := [][]string` 这里的`[][]string`可以分解成`[] []string来看`，比如`[]int`表示元素都是整型的slice，`[]string`表示元素都是字符串型的slice，那么`[][]string`就表示元素为`[]string`类型的slice

!!! warning "注意逗号"
	可以看到，golang的数据结构，当表示多个元素的时候，若使用了逗号，则最后一个元素后也必须加逗号，否则会报错
