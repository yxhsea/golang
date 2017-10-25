虽然slice不像array是永久不变的长度和容量，但是每次声明或赋值后容量是固定的。

slice扩容有多种方法

1. literal赋值，但每次赋值之后，原来的元素就丢失了，详见[关于literal](/datatype/arr_sli_key/#literal)

2. 使用append，详见[append添加元素](/datatype/arr_sli_append/)
