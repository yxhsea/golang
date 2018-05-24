interface的使用要满足2个条件才有意义：

1. 实现了interface的几个struct是相似关系（比如docker和kvm都是虚拟机）、平级的，并且输入输出参数完全一致。（这点是interface的本质，能实现interface的肯定是满足这个条件）

2. 在业务逻辑上，调用实现interface的struct是不确定的，是通过某种方式传递进来，而不是顺序的业务逻辑，比如structA、structB、structC如果是有顺序的则是错误的，下面这样是错误的：

```go
func main() {
    var i interfaceX
    i = &structA{...}
    i.Add()
    i = &structB{...}
    i.Add()
    i = &structC{...}
    i.Add()
}
```

这样逻辑是正确的：

```go
var i interfaceX
switch opt {
case "A":
    i = &structA{}
case "B":
    i = &structB{}
case "C":
    i = &structC{}
}
i.Add()
i.Del()
```

就是说调用者对于实现interface的struct是根据某个参数（通过API传递过来，或者配置文件传递过来，或者etcd传递过来）来选择某个struct，这种逻辑才适用interface。而如果程序逻辑是被调用者依次执行，则不适用interface。

总结适用interface的调用者业务逻辑（伪代码）：

```go
type I interface {
    ...
}

var i I
switch opt {    //opt通过某种方式传递进来，而不是写死
case "A":
    i = &structA{...}
case "B":
    i = &structB{...}
case "C":
    i = &structC{...}
default:
    errors.New("not support")
}
```

!!! note
	interface使用起来有无数种变形方式，但无论是那种，都要符合上面说的平行选一的业务逻辑。比如[golang-targz脑图](/attachment/golang_targz.itmz)（点击下载，并使用iThoughtsX打开），其中archive/tar/writer.go里

	```text
	func NewWriter(w io.Writer) *Writer { return &Writer{w: w} }
	```

	这里用`w io.Writer`就是因为有多种压缩方式，有gzip、有bzip2等等许多种，但无论哪种，都实现了io.Writer
