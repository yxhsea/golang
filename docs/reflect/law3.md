**To modify a reflection object, the value must be settable.**

修改反射类型变量的内部值需要保证其可设置性

反射对象可以修改实际存储的被反射对象的能力。

## **可设置性由反射对象是否能寻址原始对象来决定**

---

简单理解就是反射对象如果存在的是地址（比如原始对象是指针）就是可设置的

无法设置的例子

```go
var x float64 = 3.4
v := reflect.ValueOf(x)
v.SetFloat(7.1)
```

报错

```text
panic: reflect.Value.SetFloat using unaddressable value
```

上面的代码中我们将x值做了一份拷贝传给reflect.ValueOf方法，所以传入的参数仅仅是拷贝，而不是x本身。即使我们允许v.SetFloat(7.1)能操作成功，这并不会更新x。因此没有啥意义（设计者这样认为）。所以干脆定义它非法好了

## **查询是否可设置**

---

reflect.Value有一个[CanSet()](/reflect/pkg_type_value/#canset)方法可以用来检测Value类型的可设置性（返回一个bool类型的true或false）

## **设置值**

---

若具备可设置性，接下来可以设置值，分为2步：

1. 使用Value的[Elem()](/reflect/pkg_type_value/#elem)方法来指向内部存储的值（Elem()方法返回的也是Value）

2. 使用Value的[SetInt()](/reflect/pkg_type_value/#_2)等方法来设置值

> Just keep in mind that reflection Values need the address of something in order to modify what they represent
