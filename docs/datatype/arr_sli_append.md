```text
sli = append(sli, 元素)
```

注意：在添加元素时候，{==若原本slice的容量不够了，则会自动扩大一倍cap==}，在扩大cap时候是将原来元素复制一份（而不是引用）

!!! note "命令解读"
	sli = append(sli, 元素)意思是先读取原有元素，然后cap扩容一倍，然后在末尾插入数字0这个元素，然后把插入完的赋值给sli（赋值给新变量也是可以的），因此和原有slice脱离关系。

	每一次扩容空间，都会重新申请一块区域，把旧空间里面的元素复制进来到新空间里，然后把新的元素追加进来。旧空间里面的元素等着垃圾回收。

??? note "示例"
	```go hl_lines="31"
	func printSlice(str string, s []int) {
		fmt.Printf("%s len=%d cap=%d %v\n", str, len(s), cap(s), s)
	}

	func main() {
		var s []int
		printSlice("s", s)	//s len=0 cap=0 []

		s = append(s, 0)
		printSlice("s", s) // s len=1 cap=1 [0]

		s = append(s, 1)
		printSlice("s", s) // s len=2 cap=2 [0 1]

		s = append(s, 2)
		printSlice("s", s) // s len=3 cap=4 [0 1 2]
		a := s
		a[0] = 88
		b := s
		b[0] = 888
		printSlice("a", a) // a len=3 cap=4 [888 1 2]
		printSlice("b", b) // b len=3 cap=4 [888 1 2]

		s = append(s, 3)
		a[0] = 77
		b[0] = 777
		printSlice("s", s) // s len=4 cap=4 [777 1 2 3]
		printSlice("a", a) // a len=3 cap=4 [777 1 2]
		printSlice("b", b) // b len=3 cap=4 [777 1 2]

		s = append(s, 4)
		a[0] = 66
		b[0] = 666
		printSlice("s", s) // s len=5 cap=8 [777 1 2 3 4]
		printSlice("a", a) // a len=3 cap=4 [666 1 2]
		printSlice("b", b) // b len=3 cap=4 [666 1 2]

		s = append(s, 5)
		a[0] = 55
		b[0] = 555
		printSlice("s", s) // s len=6 cap=8 [777 1 2 3 4 5]
		printSlice("a", a) // a len=3 cap=4 [555 1 2]
		printSlice("b", b) // b len=3 cap=4 [555 1 2]

		s = append(s, 6)
		a[0] = 44
		b[0] = 444
		printSlice("s", s) // s len=7 cap=8 [777 1 2 3 4 5 6]
		printSlice("a", a) // a len=3 cap=4 [444 1 2]
		printSlice("b", b) // b len=3 cap=4 [444 1 2]

		s = append(s, 7)
		a[0] = 33
		b[0] = 333
		printSlice("s", s) // s len=8 cap=8 [777 1 2 3 4 5 6 7]
		printSlice("a", a) // a len=3 cap=4 [333 1 2]
		printSlice("b", b) // b len=3 cap=4 [333 1 2]

		s = append(s, 8)
		a[0] = 22
		b[0] = 222
		printSlice("s", s) // s len=9 cap=16 [777 1 2 3 4 5 6 7 8]
		printSlice("a", a) // a len=3 cap=4 [222 1 2]
		printSlice("b", b) // b len=3 cap=4 [222 1 2]

		s = append(s, 9)
		a[0] = 11
		b[0] = 111
		printSlice("s", s) // s len=10 cap=16 [777 1 2 3 4 5 6 7 8 9]
		printSlice("a", a) // a len=3 cap=4 [111 1 2]
		printSlice("b", b) // b len=3 cap=4 [111 1 2]
	}
	```

	注意：虽然a是s的衍生出来的，但从上面背景高亮部分开始，a和s就分道扬镳了

!!! warning "建议使用make()提前申请空间"
	建议使用`make([]int64, 0, 8)`这种方式提前申请好空间，以便降低append扩容引起的旧空间内存回收问题，来达到性能优化。
