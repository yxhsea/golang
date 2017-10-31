## **检索值**

---

```text
m[key]
```

比如

```text
a := m[key]
```

!!! warning ""
	当检索不到时候，不会报错，而是a的值就是value类型的默认值，比如value是int类型，那a就是0

two-value写法：

```text
a, ok := m[key]
```

当key存在时，ok是true，否则是false


## **插入或更新值**

---

当key不存在时候就是插入新的key-value：

```text
m[key] = value
```
