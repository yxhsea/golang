1. 没有interface时候:

	```go
	package main

	import "fmt"

	type Driver struct {
		Name string
	}

	type Benz struct{}

	func (this *Benz) Drive(dr Driver) {
		fmt.Printf("%s drives Benz.\n", dr.Name)
	}

	func main() {
		me := Driver{Name: "Cyent"}

		// 开Benz
		car := &Benz{}
		car.Drive(me)
	}
	```
	
	!!! note ""
		注意上面main函数里写了多少行，这里一共有3行有效代码

	输出

	```text
	Cyent drives Benz.
	```

2. 现在增加一辆Audi

	```go
	package main

	import "fmt"

	type Driver struct {
		Name string
	}

	type Benz struct{}

	func (this *Benz) Drive(dr Driver) {
		fmt.Printf("%s drives Benz.\n", dr.Name)
	}

	type Audi struct {}

	func (this *Audi) Drive(dr Driver) {
		fmt.Printf("%s drives Audi.\n", dr.Name)
	}

	func main() {
		me := Driver{Name: "Cyent"}

		// car为外部传递进来的值，假设传进来的是Audi
		get_car := "Audi"
		switch get_car {
		case "Benz":
			car := &Benz{}
			car.Drive(me)
		case "Audi":
			car := &Audi{}
			car.Drive(me)
		}
	}
	```

	注意：

	1. 由于case里是局部变量作用域，因此`car.Drive(me)`只能放在case里面，无法放在switch外（python是可以的）。因此这种写法也不友好

	2. 注意上面main函数里用了switch可以对传递进来的变量做判断，但这么写也不够好，因为每增加一辆车，switch都要多加一个case，那有没有无论增加多少辆车，main函数永远不用改代码的办法呢，当然有，接下来该interface上场了

3. 使用interface

	```go
	package main

	import (
		"fmt"
		"errors"
	)

	type Car interface {
		Drive(Driver)
	}

	func NewCar(c string) (Car, error) {
		switch c {
		case "Benz":
			return &Benz{}, nil
		case "Audi":
			return &Audi{}, nil
		default:
			return nil, errors.New("not support")
		}

		return nil, errors.New("not support")
	}

	type Driver struct {
		Name string
	}

	type Benz struct{}

	func (this *Benz) Drive(dr Driver) {
		fmt.Printf("%s drives Benz.\n", dr.Name)
	}

	type Audi struct {}

	func (this *Audi) Drive(dr Driver) {
		fmt.Printf("%s drives Audi.\n", dr.Name)
	}

	func main() {
		me := Driver{Name: "Cyent"}

		// 外部传递进来的值，假设传进来的是Audi
		car, err := NewCar("Audi")
		if err != nil {
			fmt.Println(err)
		} else {
			car.Drive(me)
		}
	}
	```

	输出

	```text
	Cyent drives Audi.
	```

	注意: 上面main函数里没有明文提到具体什么车，因此无论增加多少辆车，main函数里的代码可以长久不变，这就是将开车和具体开什么车在main函数里做了解耦
