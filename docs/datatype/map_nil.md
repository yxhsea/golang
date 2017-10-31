只有仅写`var m map[key_type]value_type`才能得到nil map

一旦初始化，即`m = make(map[key_type]value_type)`就不是nil map了

!!! note
	nil map是无法通过%T看到，%T看到的都是map[key_type]value_type，比如map[string]int

	若往nil map（即未make的map）中插入键值对，则会报错`panic: assignment to entry in nil map`
