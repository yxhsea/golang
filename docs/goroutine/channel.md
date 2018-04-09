## **channel**

---

channel是有类型的管道，可以用channel操作符 `<-` 对其发送或者接收值。

## **创建channel**

---

```text
c := make(chan int)
```

注意：管道必须指定类型，比如这里是int，即往管道里传送的数据只能是int类型。当然可以是interface{}这种空接口方式，这样就可以传送各种数据类型了

## **管道方向**

---

```text
ch <- v // Send v to channel ch.
v := <-ch // Receive from ch, and
           // assign value to v.
```

“箭头”就是数据流的方向

**注意接受时候<-和chan变量之间没有空格，虽然有空格也不会报错，但是ide会提示，因此还是依照规范不空格比较好**
