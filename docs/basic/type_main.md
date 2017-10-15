```go
// Bool类型
var my_bool bool = true

// 字符串类型
var my_string string = "hello, world!"

// int: 有符号整形，根据系统架构自动判断是int8，int16，int32还是int64
// 比如当前系统是64位系统，则为int64
var my_int_min int = -9223372036854775808
var my_int_max int = 9223372036854775807

// int8: 有符号 8 位整型 (-128 到 127)
var my_int8_min int8 = -128
var my_int8_max int8 = 127

// int16: 有符号 16 位整型 (-32768 到 32767)
var my_int16_min int16 = -32768
var my_int16_max int16 = 32767

// int32: 有符号 32 位整型 (-2147483648 到 2147483647)
var my_int32_min int32 = -2147483648
var my_int32_max int32 = 2147483647

// int64: 有符号 64 位整型 (-9223372036854775808 到 9223372036854775807)
var my_int64_min int64 = -9223372036854775808
var my_int64_max int64 = 9223372036854775807

// uint: 无符号整形，根据系统架构自动判断是uint8，uint16，uint32还是uint64
// 比如当前系统是64位系统，则为uint64
var my_uint_min uint = 0
var my_uint_max uint = 18446744073709551615

// uint8: 无符号 8 位整型 (0 到 255)
var my_uint8_min uint8 = 0
var my_uint8_max uint8 = 255

// uint16: 无符号 16 位整型 (0 到 65535)
var my_uint16_min uint16 = 0
var my_uint16_max uint16 = 65535

// uint32: 无符号 32 位整型 (0 到 4294967295)
var my_uint32_min uint32 = 0
var my_uint32_max uint32 = 4294967295

// uint64: 无符号 64 位整型 (0 到 18446744073709551615)
var my_uint64_min uint64 = 0
var my_uint64_max uint64 = 18446744073709551615

// uintptr: 无符号整型，用于存放一个指针，可以足够保存指针的值的范围。和uint范围相同，根据系统架构自动判断
var my_uintptr_min uintptr = 0
var my_uintptr_max uintptr = 18446744073709551615

// byte: uint8的别名
var my_byte_min byte = 0
var my_byte_max byte = 255

// rune: int32的别名。代表1个unicode码
var my_rune_min rune = -2147483648
var my_rune_max rune = 2147483647

// float32: 单精度浮点数，在C语言里等同于float
// float64: 双精读浮点数，在C语言里等同于double
// 如果不写类型，则为float64（暂时不知道这个是根据系统架构判断还是默认就是float64）
// 从结果可以看出:
//   float32只能容纳8位数字（包括小数点前后数字，不包括小数点，超过8位会将四舍五入保留8位）
//   float64可以容纳比较多的数字（具体暂时还没测），而且这种双精度我也一直没搞懂，很复杂
//   当符合要求时候会自动用科学计数法来表示，要注意
var my_float32 float32 = 10086.141592653
var my_float64 float64 = 10086.141592653
var my_float_auto = 10086.141592653
```

输出

```text
bool: true
string: hello, world!
int min: -9223372036854775808
int max: 9223372036854775807
int8 min: -128
int8 max: 127
int16 min: -32768
int16 max: 32767
int32 min: -2147483648
int32 max: 2147483647
int64 min: -9223372036854775808
int64 max: 9223372036854775807
uint min: 0
uint max: 18446744073709551615
uint8 min: 0
uint8 max: 255
uint16 min: 0
uint16 max: 65535
uint32 min: 0
uint32 max: 4294967295
uint64 min: 0
uint64 max: 18446744073709551615
uintptr min: 0
uintptr max: 18446744073709551615
byte min: 0
byte max: 255
rune min: -2147483648
rune max: 2147483647
float32: 10086.142
float64: 10086.141592653
float_auto: 10086.141592653
```

除了上述的数据类型，还有2个复数：complex64、complex128

!!! note "-0和0"
	-0等于0，用-0能编译通过，实际输出就是0。无论是unit还是int

!!! note "int8"
	int8意思是用二进制表示的话，从左往右第1位表示正数或是负数，第1位为0表示正数，1表示负数

	为什么int8最小是-128而最大是127，感觉不对称，但是正确的。是因为00000000和10000000都可以表示0，但是没必要存在2个0，因此往负数方向移1位，即00000000表示0，而10000000表示为-1，那么11111111就表示为-128。
