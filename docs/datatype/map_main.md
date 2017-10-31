map就是键值对(key->value)，在其他语言中通常叫hash或者字典

## **完整写法**

---

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

## **短声明写法**

---

```text
m := make(map[key_type]value_type)
```

例子

```text
m := make(map[string]int)
```
