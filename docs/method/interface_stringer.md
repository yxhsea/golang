在fmt包里有一个interface叫做Stringer：

```text
type Stringer interface {
    String() string
}
```

作用：fmt.Println或打印一个变量的值的时候，会判断这个变量是否实现了Stringer接口，如果实现了，则调用这个变量的String()方法，并将返回值打印到屏幕上

!!! note ""
	fmt.Printf的%v也会读取Stringer

例子：

```go
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

func (p Person) String() string {
	return fmt.Sprintf("(Name: %v) (Age: %v)", p.Name, p.Age)
}

func main() {
	a := Person{"benz", 21}
	fmt.Println(a)
	fmt.Printf("%v\n", a)
}
```

输出

```text
(Name: benz) (Age: 21)
(Name: benz) (Age: 21)
```

可以看出，用String()修改了输出格式

再举个例子：

```go
package main

import "fmt"

type IPAddr [4]byte

/*
func (ip IPAddr) String() string {
	return fmt.Sprintf("%v.%v.%v.%v", ip[0], ip[1], ip[2], ip[3])
}
*/

func main() {
	hosts := map[string]IPAddr{
		"loopback":  {127, 0, 0, 1},
		"googleDNS": {8, 8, 8, 8},
	}
	for name, ip := range hosts {
		fmt.Printf("%v: %v\n", name, ip)
	}
}
```

输出

```text
loopback: [127 0 0 1]
googleDNS: [8 8 8 8]
```

如果把上面的String的注释去掉，输出则是

```text
loopback: 127.0.0.1
googleDNS: 8.8.8.8
```
