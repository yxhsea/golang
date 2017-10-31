## **分行必须有逗号**

---

golang有一个特点就是如果写一行，在最后一个元素后面可以逗号也可以不逗号，但如果是分行写，就必须有逗号，比如

```text
m[Key{"A","B"}] = Value{1,2}
m[Key{"A","B"}] = Value{
	1, 2,
}
```

注：只有以下这种情况才不需要逗号，如

```text
import (
	"fmt"
	"math"
)

type (
	MyInt int
	AAInt int
)
```
