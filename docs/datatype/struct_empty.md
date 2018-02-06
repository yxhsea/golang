空struct这么表示:

```text
struct{}{}
```

例子:

```go
package main

import "fmt"

func main() {
	s := struct{}{}
	fmt.Printf("%T,%v,%+v\n", s, s, s)
}
```

输出

```text
struct {},{},{}
```

!!! note
	大括号左右可以多个空格，比如这样也是可以的

	```text
	struct  {} {}
	```
