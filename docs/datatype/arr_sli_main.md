在golang中有array（数组）和slice（切片）关系紧密，因此放在一个章节里阐述

下面将array和slice从不同维度进行比较

## **声明赋值**

---

array

```go
arr := [6]int{2, 3, 5, 7, 11, 13}
```

slice

```go
sli := arr[1:4]
```

!!! note ""
	arr[1:4]就是对arr做的切片

## **本质**

---

array就是array

slice是array元素的引用，官方把slice叫做underlying array、Slices are like references to arrays

!!! note ""
	从使用效果上看，slice完全就像是array元素的指针

## **长度是否可变**

---

array是固定长度，占用内存中固定的一块地址，因此不可能删除array元素来导致长度变化

slice是可变长度，内存地址可扩展

## **2种声明方式**

---

array

- 普通方式：

	```go
	//显式声明数组arr，长度为6，元素类型为int
	var arr [6]int
	arr[0] = 2
	arr[1] = 3
	arr[2] = 5
	arr[3] = 7
	arr[4] = 11
	arr[5] = 13
	```

- literal方式：

	```go
	arr := [6]int{2, 3, 5, 7, 11, 13}
	```

slice

- 普通方式：

	```go
	//显式声明切片sli，其中可见元素为arr的第2个元素地址到第4个元素地址，但容量为第2个元素地址到最后一个元素地址
	var sli []int = arr[1:4]
	```

- literal方式：

	```gp
	sli := []int{2, 3, 5, 7, 11, 13}
	```

## **类型**

---

array的类型示例：[6]int

slice的类型示例：[]int

因此slice也叫做数组类型（这个忘了为啥叫数组类型）
