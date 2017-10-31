```go
type Vertex struct {
	Lat, Long float64
}

var m = map[string]Vertex{
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": Vertex{
		37.42202, -122.08408,
	},
}

func main() {
	fmt.Println(m)
	fmt.Printf("%T\n", m)
}
```

输出

```text
map[Bell Labs:{40.68433 -74.39967} Google:{37.42202 -122.08408}]
map[string]main.Vertex
```

!!! note "简写方式"
	> If the top-level type is just a type name, you can omit it from the elements of the literal.

	```go
	var m = map[string]Vertex{
		"Bell Labs": Vertex{
			40.68433, -74.39967,
		},
		"Google": Vertex{
			37.42202, -122.08408,
		},
	}
	```
	可以简写为

	```go
	var m = map[string]Vertex{
		"Bell Labs": {40.68433, -74.39967},
		"Google":    {37.42202, -122.08408},
	}
	```
