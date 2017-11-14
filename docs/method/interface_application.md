interface的使用要满足2个条件才有意义：

1. 实现了interface的几个struct是相似关系（比如docker和kvm都是虚拟机）、平级的，并且输入输出参数完全一致。（这点是interface的本质，能实现interface的肯定是满足这个条件）

2. 在业务逻辑上，调用实现interface的struct是不确定的，是通过某种方式传递进来，而不是顺序的业务逻辑，比如
这样是不对的，因为netns_struct、bridge_struct、veth_struct是有顺序的（先有netns、bridge，才能有veth）：

```go
func main() {
    var i vnic
    i = &netns_struct{...}
    i.Add()
    i = &bridge_struct{...}
    i.Add()
    i = &veth_struct{...}
    i.Add()
}
```

这样逻辑是正确的：

```go
var i vnic
switch opt {
case "netns":
    i = &netns.struct{}
case "bridge":
    i = &bridge.struct{}
case "veth":
    i = &veth.struct{}
}
i.Add()
i.Del()
```

就是说调用者这方对于实现interface的struct是根据某个参数（通过API传递过来，或者配置文件传递过来，或者etcd传递过来）来选择某个struct，这种逻辑才适用interface。而在vrouter程序逻辑里，netns、bridge、veth是被调用者依次执行，因此不适用interface。

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
