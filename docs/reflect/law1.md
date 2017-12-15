**Reflection goes from interface value to reflection object.**

反射可以从接口值到反射对象

!!! note
	这里反射类型对象就是指通过TypeOf()或者ValueOf()得到的反射类型的变量（即reflect.\*rtype或reflect.Value）

> At the basic level, reflection is just a mechanism to examine the type and value pair stored inside an interface variable.

基本来说，反射仅仅是一种校验接口变量hold的变量的类型和值(pair)的机制

!!! note "关于pair的理解"
	详见[接口-深入理解-pair](/method/interface_deep/#pair)

我的理解，就是可以通过TypeOf()、ValueOf()得到反射类型的变量，然后可以通过调用这个变量的方法来得到相应的信息，比如通过Kind()方法可以得到基类型，通过Int()方法可以用于得到int*类型的值(int64)

!!! warning
	无法通过反射知道一个变量是否为接口变量，用`switch i.(type); case`这种类型判断也是无法知道的，详见[type switch章节](/method/interface_typeswitch/#_1)
