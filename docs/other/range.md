range主要用于for循环，用来迭代取出slice或map的元素

用于slice时候，会返回2个值：

1. 第一个值是元素下标，比如第一个元素的下标是0

2. 第二个值是元素内容

!!! warning ""
	对slice用range时候取的是可见元素，即不超过len范围，而不是cap元素。因为可见元素才有值，cap只是容量，超出可见部分是没有值的。

举例

```go
pow := []int{2, 3, 5, 7, 11, 13}
for i, v := range pow {
    fmt.Printf("index=%d, value=%d\n", i, v)
}
```

输出

```text
index=0, value=2
index=1, value=3
index=2, value=5
index=3, value=7
index=4, value=11
index=5, value=13
```

可以用下划线_作为占位符，表示不需要取值

如果只想取index，不想取value，可以不写`, value`，比如

```go
pow := make([]int, 10)
for i := range pow {
	pow[i] = 1 << uint(i) // == 2**i
}
for _, value := range pow {
	fmt.Printf("%d\n", value)
}
```

输出

```text
1
2
4
8
16
32
64
128
256
```

!!! note "关于左移和右移运算符"
	[左移运算符](/other/binary_operation/#_1)

	[右移运算符](/other/binary_operation/#_2)
