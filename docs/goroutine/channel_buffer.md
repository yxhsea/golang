**Buffered Channels**

channel可以带buffer：

```text
make(chan int, 2)
```

表示创建一个length为2的管道，注意这里的length表示数量，而非具体的内存容积，比如

```text
make(chan string, 2)
```

- 表示可以容纳2个string的内容，比如"abcdefg"和"h"这样2个string，而不是指只能存"a"和"b"。

- 当不设置length时候即为buffer=0

说明：

- 往chan发送数据时候，若buffer没满的时候，则发送数据即刻成功，不会被阻塞。当buffer满了就会被阻塞

- 从chan接受数据时候，若buffer是空的则会被阻塞，当buffer不是空的时候则即刻完成，不会被阻塞
