## **io.Reader**

---

io 包指定了 io.Reader 接口， 它表示从数据流结尾读取。

Go标准库包含这个了io.Reader接口的许多实现，包括文件、网络连接、压缩、加密等

io.Reader接口的定义：

```go
type Reader interface {
	Read(p []byte) (n int, err error)
}
```

任意的type只要实现了`Read(p []byte) (n int, err error)`就可以被其他程序通过实现Reader接口方式来调用Read()方法

Read()：用数据填充指定的字节 slice，并且返回填充的字节数和错误信息。 在遇到数据流结尾时，返回 io.EOF 错误。

## **strings.Reader**

---

strings.Reader是个struct，定义如下

```go
// A Reader implements the io.Reader, io.ReaderAt, io.Seeker, io.WriterTo,
// io.ByteScanner, and io.RuneScanner interfaces by reading
// from a string.
type Reader struct {
	s        string
	i        int64 // current reading index
	prevRune int   // index of previous rune; or < 0
}
```

strings.Reader实现了io.Reader，因为strings.Reader拥有符合io.Reader要求的Read()方法，定义如下

```go
func (r *Reader) Read(b []byte) (n int, err error) {
	if r.i >= int64(len(r.s)) {
		return 0, io.EOF
	}
	r.prevRune = -1
	n = copy(b, r.s[r.i:])
	r.i += int64(n)
	return
}
```

下面例子代码创建了一个 strings.Reader。 并且以每次 8 字节的速度读取它的输出

```go
package main

import (
	"fmt"
	"io"
	"strings"
)

func main() {
	r := strings.NewReader("Hello, Reader!")

	b := make([]byte, 8)
	for {
		n, err := r.Read(b)
		fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
		fmt.Printf("b[:n] = %q\n", b[:n])
		if err == io.EOF {
			break
		}
	}
}
```

输出：

```text
n = 8 err = <nil> b = [72 101 108 108 111 44 32 82]
b[:n] = "Hello, R"
n = 6 err = <nil> b = [101 97 100 101 114 33 32 82]
b[:n] = "eader!"
n = 0 err = EOF b = [101 97 100 101 114 33 32 82]
b[:n] = ""
```

注意点：

1. r.Read(b) 表示读取r里面的字符串（即Hello, Reader!），每次读出8个字节并存入slice b中，存入方式是从第一个元素开始存，不管元素原来是否已经有值都直接覆盖。因此要做EOF的判断，因为最终slice的值是最后一次循环读出的（靠前）+ 最后一次的前一次循环读出的（靠后），即假设字符串是从a到z，而且[]byte, 3，那么最终slice的值类似这样[x, y, z, u, v, w]

2. 每次r.Read()之后已经被读取的r里面的字符串就不能被重新读取了

如果上面例子不是很懂，对上面例子稍微改造下，更详细的输出

```go
package main

import (
	"fmt"
	"io"
	"strings"
)

func main() {
	r := strings.NewReader("Hello, Reader!")

	b := make([]byte, 4)
	fmt.Println("===init===")
	fmt.Println("r:", r)
	fmt.Println("b:", b)

	fmt.Println("===for begin===")
	for {
		n, err := r.Read(b)
		fmt.Println("r:", r)
		fmt.Printf("n = %v, err = %v, b = %v\n", n, err, b)
		fmt.Printf("b[:n] = %q\n", b[:n])
		if err == io.EOF {
			break
		}
		fmt.Println("---")
	}

	fmt.Println("===end===")
	n, err := r.Read(b)
	fmt.Println("r:", r)
	fmt.Printf("n = %v, err = %v, b = %v\n", n, err, b)
	fmt.Printf("b[:n] = %q\n", b[:n])
}
```

输出：

```text
===init===
r: &{Hello, Reader! 0 -1}
b: [0 0 0 0]
===for begin===
r: &{Hello, Reader! 4 -1}
n = 4, err = <nil>, b = [72 101 108 108]
b[:n] = "Hell"
---
r: &{Hello, Reader! 8 -1}
n = 4, err = <nil>, b = [111 44 32 82]
b[:n] = "o, R"
---
r: &{Hello, Reader! 12 -1}
n = 4, err = <nil>, b = [101 97 100 101]
b[:n] = "eade"
---
r: &{Hello, Reader! 14 -1}
n = 2, err = <nil>, b = [114 33 100 101]
b[:n] = "r!"
---
r: &{Hello, Reader! 14 -1}
n = 0, err = EOF, b = [114 33 100 101]
b[:n] = ""
===end===
r: &{Hello, Reader! 14 -1}
n = 0, err = EOF, b = [114 33 100 101]
b[:n] = ""
```
