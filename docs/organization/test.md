go内置一个轻量级测试框架，使用方法：

1. 文件以`_test.go`结尾

2. `import "testing"`

3. 函数名为`TestXXX`，并且parameter为`t *testing.T`

例子：

在`$GOPATH/src/github.com/cyent/golang/example/stringutil`里新建文件`reverse_test.go`

```go
package stringutil

import "testing"

func TestReverse(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := Reverse(c.in)
		if got != c.want {
			t.Errorf("Reverse(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
```

执行测试：

```text
go test github.com/cyent/golang/exmaple/stringutil
```

或者在`$GOPATH/src/github.com/cyent/golang/exmaple/stringutil`目录下执行`go test`

输出

```text
ok  	github.com/cyent/golang/example/stringutil	0.006s
```

表示测试通过

如果是在stringutil目录下直接执行`go test`，会输出

```text
PASS
ok  	github.com/cyent/golang/example/stringutil	0.005s
```
