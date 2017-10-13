## **创建lib**

---

创建`$GOPATH/src/github.com/cyent/golang/example/stringutil`目录，并在目录里放置reverse.go，内容如下

```go
// Package stringutil contains utility functions for working with strings.
package stringutil

// Reverse returns its argument string reversed rune-wise left to right.
func Reverse(s string) string {
	r := []rune(s)
	for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}
```

```text
go install github.com/cyent/golang/example/stringutil
```

会看到生成了`$GOPATH/pkg/darwin_amd64/github.com/cyent/golang/example/stringutil.a`

!!! warning "注意这里的路径"
	路径不是`cyent/golang/example/stringutil/stringutil.a`，而是`cyent/golang/example/stringutil.a`。

	上面`reverse.go`里的代码也可以除了放在`$GOPATH/src/github.com/cyent/golang/example/stringutil/reverse.go`，也可以放在`$GOPATH/src/github.com/cyent/golang/example/stringutil.go`。实际效果是一样的。

	用stringutil目录只是为了分隔代码，这样比较容易让人理解，而且如果stringutil包里的函数比较多的时候，就需要分成多个源码文件，用文件名来标识内容，提高可读性，比如函数名叫做Reverse，文件名就叫做reverse.go。如果所有函数都放在一个代码文件里，那么这个代码文件就太庞大，显得很乱，不利于维护

## **调用lib**

修改[上一页](/organization/helloworld/#hello-world)中的`hello.go`，改为：

```go
package main

import (
	"fmt"
	"github.com/cyent/golang/example/stringutil"
)

func main() {
	fmt.Println(stringutil.Reverse("!oG ,olleH"))
}
```

执行

```text
go install github.com/cyent/golang/example/hello
```

就会编译出可执行的二进制程序并自动放入`$GOPATH/bin`中，比如这个例子中生成的程序就是`$GOPATH/bin/hello`

执行

```text
$GOPATH/bin/hello
```

效果

```text
Hello, Go!
```

!!! warning "注意生成的二进制路径"
	这个hello二进制文件生成路径是`$GOPATH/bin/hello`，而不是`$GOPATH/bin/github.com/cyent/golang/example/hello`，说明一个workspace生成的二进制文件都在一起

	一个workspace里可执行程序名不能相同，因为都是放在bin目录下，这目录下没有任何子目录。即不能存在这样2个main package在一个workspace里：（假设下面2个hello.go都是package main）
	$GOPATH/src/github.com/cyent/example/repo1/hello.go
	$GOPATH/src/github.com/cyent/example/repo2/hello.go

!!! note "自动搜索所有函数并集中到一个.a文件"
	生成library时候是会自动把`$GOPATH/src/github.com/cyent/golang/example/stringutil/`下面所有.go文件里包含的函数集中到`stringutil.a`中（根据这些.go文件中的package xxx来自动加入到一个.a），因此如果存在相同的函数，会报错提示重复

!!! warning "$GOPATH/pkg路径"
	注意`go install`会将`stringutil.a`对象放到`pkg/darwin_amd64`目录中，它会反映出其源码目录。 这就是在此之后调用go工具，能找到包对象并避免不必要的重新编译的原因。darwin_amd64这部分能帮助跨平台编译，并反映出你的操作系统和架构。

!!! note "静态链接"
	Go的可执行命令是静态链接的；在运行生成的可执行二进制程序时，包对象`stringutil.a`无需存在。

!!! warning "go源码文件第一行必须是package name"
	package name就是调用时候import中/最后一段，比如想别的程序通过import "crypto/rot13"调用，那么就写package rot13

	生成可执行程序，package必须为main

## **附: 此时GOPATH目录结构**

---

```text
.
├── bin
│   └── hello
├── pkg
│   └── darwin_amd64
│       └── github.com
│           └── cyent
│               └── golang
│                   └── example
│                       └── stringutil.a
└── src
    └── github.com
        └── cyent
            └── golang
                └── example
                    ├── hello
                    │   └── hello.go
                    └── stringutil
                        └── reverse.go
```
