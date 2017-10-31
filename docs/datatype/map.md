map就是键值对(key->value)，在其他语言中通常叫hash或者字典

## **声明**

---

1. 完整写法

	```text
	var m map[key_type]value_type
	m = make(map[key_type]value_type)
	```

	例子

	```text
	var m map[string]int
	m = make(map[string]int)
	```

	!!! note "简写"
		```text
		var m map[key_type]value_type = make(map[key_type]value_type)
		```

		例子

		```text
		var m map[string]int = make(make[string]int)
		```

2. 短声明写法

	```text
	m := make(map[key_type]value_type)
	```

	例子

	```text
	m := make(map[string]int)
	```

## **nil map**

---

只有仅写`var m map[key_type]value_type`才能得到nil map，一旦初始化，即`m = make(map[key_type]value_type)`就不是nil map了

!!! note
	nil map是无法通过%T看到，%T看到的都是map[key_type]value_type，比如map[string]int

	若往nil map（即未make的map）中插入键值对，则会报错`panic: assignment to entry in nil map`

## **key和value可以用struct**

---

```go
type Key struct {
    a, b string
}

type Value struct {
    c, d int
}

var m map[Key]Value

func main() {
    m = make(map[Key]Value)
    m[Key{
        "A", "B",
    }] = Value{
        1, 2,
    }
    fmt.Println(m[Key{
        "A","B",
    }])
    fmt.Println(m)
    fmt.Printf("%T\n", m)
}
```

输出

```text
{1 2}
map[{A B}:{1 2}]
map[main.Key]main.Value
```

!!! note
	1. Key、Value写在函数外和写在main函数里，都是main.key、main.Value

	2. 上面的Key、Value、m的声明都可以放到main函数里写
