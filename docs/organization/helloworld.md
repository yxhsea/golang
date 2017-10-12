## **程序入口main.main**

---

```go
package main

func main() {
    ...
}
```

## **hello, world**

---

在一个package里创建一个`xxx.go`，比如在`$GOPATH/src/github.com/cyent/golang/example/hello`里创建了一个myfirst.go（也可以叫其他文件名，重点是该文件里包含着main.main），里面写着：

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, world!")
}
```

然后在任意目录下执行
```text
go install github.com/cyent/golang/example/hello
```

就会编译出可执行的二进制程序并自动放入`$GOPATH/bin`中，比如这个例子中生成的程序就是`$GOPATH/bin/hello`

!!! warning "注意生成的二进制文件名"
	可以看出来，生成的二进制文件名是根据package名自动生成的（即目录名: hello），和文件名myfirst.go毫无关系

!!! note "go的规范是目录名和文件名一致"
	通常是`$GOPATH/src/github.com/cyent/golang/example/hello/hello.go`，这样比较符合习惯，也是go的编码规范

执行

```text
$GOPATH/bin/hello
```

!!! note ""
	如果有将`$GOPATH/bin`加入到$PATH里，那么这里直接执行hello即可

效果

```text
Hello, world!
```

执行`go install`还可以在package目录下执行，这样就不用跟package路径了。比如在`$GOPATH/src/github.com/cyent/golang/example/hello`目录下直接执行`go install`即可

## **附: 此时GOPATH目录结构**

---

```text
.
├── bin
│   └── hello
├── pkg
└── src
    └── github.com
        └── cyent
            └── golang
                └── example
                    └── hello
                        └── hello.go
```
