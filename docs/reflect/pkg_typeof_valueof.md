## **reflect.TypeOf()**

---

用于从接口值中检索reflect.Type，函数返回reflect.\*rtype（实现了reflect.Type）

??? note "reflect.TypeOf在go/src/reflect/type.go里的定义"
	```go
	// TypeOf returns the reflection Type that represents the dynamic type of i.
	// If i is a nil interface value, TypeOf returns nil.
	func TypeOf(i interface{}) Type {
	    eface := *(*emptyInterface)(unsafe.Pointer(&i))
	    return toType(eface.typ)
	}
	```

## **reflect.ValueOf()**

---

用于从接口值中检索reflect.Value，函数返回reflect.Value

??? note "reflect.ValueOf在go/src/reflect/value.go里的定义"
	```go
	// ValueOf returns a new Value initialized to the concrete value
	// stored in the interface i. ValueOf(nil) returns the zero Value.
	func ValueOf(i interface{}) Value {
		if i == nil {
			return Value{}
		}

		// TODO: Maybe allow contents of a Value to live on the stack.
		// For now we make the contents always escape to the heap. It
		// makes life easier in a few places (see chanrecv/mapassign
		// comment below).
		escapes(i)

		return unpackEface(i)
	}
	```
