## **什么是回调函数**

---

关于回调的理解，知乎上有个不错，这里贴出来: https://www.zhihu.com/question/19801131

函数值可用于回调，下面是关于回调函数的理解:

回调函数英文叫做callback，技术本质是就是一个通过函数指针（只不过在golang里叫函数值，不叫函数指针）调用的函数。比如将函数A作为参数传递给函数B，那么A就叫做 ***回调函数*** ，B就叫做 ***中间函数***，调用B的函数叫做 ***起始函数***

!!! note
	中间函数：什么时间、做什么

	回调函数：怎么做

回调示例（来自知乎）：

- 有一家旅馆提供叫醒服务，但是要求旅客自己决定叫醒的方法。可以是打客房电话，也可以是派服务员去敲门，睡得死怕耽误事的，还可以要求往自己头上浇盆水。这里，“叫醒”这个行为是旅馆提供的，相当于中间函数，但是叫醒的方式是由旅客决定并告诉旅馆的，也就是回调函数。而旅客告诉旅馆怎么叫醒自己的动作，也就是把回调函数传入中间函数的动作，称为登记回调函数（to register a callback function）

- 你到一个商店买东西，刚好你要的东西没有货，于是你在店员那里留下了你的电话，过了几天店里有货了，店员就打了你的电话，然后你接到电话后就到店里去取了货。在这个例子里， 你的电话号码就叫 ***回调函数*** ，你把电话留给店员就叫 ***登记回调函数*** ，店里后来有货了叫做 ***触发了回调关联的事件*** ，店员给你打电话叫做 ***调用回调函数*** ，你到店里去取货叫做 ***响应回调事件***  。

!!! note "回调: 回过头来调用"
	回调函数这4个字容易让人理解为调用别人，这是错误的，如果在前面加上一个"被"字就好理解了，即等着被中间函数调用，只不过回调函数是主程序写的，也就是自己的意思，所以叫回调。

	我是这么记忆的：我把我写的函数，传递给你，希望你在某个时候 **回过头来调用** 我的这个函数。那么这个函数就称为回调函数

下面是一个代码例子，可以比较下python和golang在回调方面的编码区别

- python示例

	??? note "even.py"
		```python
		#!/usr/bin/env python
		# -*- coding: utf-8 -*-

		#回调函数1
		#生成一个2k形式的偶数
		def double(x):
		    return x * 2

		#回调函数2
		#生成一个4k形式的偶数
		def quadruple(x):
		    return x * 4
		```

	??? note "main.py"
		```python
		from even import *

		#中间函数
		#接受一个生成偶数的函数作为参数
		#返回一个奇数
		def getOddNumber(k, getEvenNumber):
		    return 1 + getEvenNumber(k)

		#起始函数，这里是程序的主函数
		def main():
		    k = 1

		    #当需要生成一个2k+1形式的奇数时
		    i = getOddNumber(k, double)
		    print(i)

		    #当需要一个4k+1形式的奇数时
		    i = getOddNumber(k, quadruple)
		    print(i)

		if __name__ == "__main__":
		    main()
		```

	```text
	> python main.py
	> 3
	> 5
	```

- golang示例

	??? note "$GOPATH/src/github.com/cyent/golang/example/even/even.go"
		```go
		package even

		//回调函数1
		//生成一个2k形式的偶数
		func Double(x int) int {
			return x * 2
		}

		//回调函数2
		//生成一个4k形式的偶数
		func Quadruple(x int) int {
			return x * 4
		}
		```

	??? note "main.go"
		```go
		package main

		import (
			"fmt"
			"github.com/cyent/golang/example/even"
		)

		//中间函数
		//接受一个生成偶数的函数作为参数
		//返回一个奇数
		func getOddNumber(k int, getEvenNumber func(int) int) int {
			return 1 + getEvenNumber(k)
		}

		//起始函数，这里是程序的主函数
		func main() {
			k := 1

			//当需要生成一个2k+1形式的奇数时
			i := getOddNumber(k, even.Double)
			fmt.Println(i)

			//当需要一个4k+1形式的奇数时
			i = getOddNumber(k, even.Quadruple)
			fmt.Println(i)
		}
		```

	```text
	> go run main.go
	> 3
	> 5
	```

可以看出，python是隐式函数指针，而golang是显式函数指针（函数值）

## **我的例子**

---

使用libvirt api对虚拟机进行操作，有2种方式：

1. 只写一个libvirt模块，接收参数来对虚拟机进行启动、停止、重启、删除等操作。但这样有3个问题
	- 这个模块要做很多类型的事情，包括连接libvirt接口、宿主机操作（网络、存储、性能获取）、vm操作（创建、删除、启动、停止）
	- 这个模块要维护libvirt版本的兼容性，当版本越来越多的时候，会因为兼容各个版本导致模块很臃肿
	- 每增加一个对虚拟机操作的功能，都是在更新这个模块，次数多了，自然也变得臃肿

2. libvirt模块只写最基础的功能，例如只写连接libvirt接口，然后接收一个函数来对宿主机或者虚拟机做操作。然后调用者自己实现宿主机操作的回调函数、虚拟机操作的回调函数，例如需要对某个虚拟机进行启动，操作是：
	1. 调用libvirt模块连接api
	2. 传入调用者自己实现的启动虚拟机的回调函数作为参数传递给libvirt模块

	这样的好处有2个：

	- 将原本libvirt模块需要做的3件事，拆解开来，维护量大大减少
	- 调用者需要哪个功能，就实现哪个功能，不需要就不用实现，代码开量明显减少

	例如：

	上面提到的调用者，可以叫vm.py，libvirt模块分为kvm.py、container.py，用到kvm.py时候就import kvm

	vm.py
	```python
	import kvm

	def reboot(dom_uuid, host_ip):
		def shutdown_vm(dom):
			...
		kvm.libvirt_connect(shutdown_vm(dom), host_ip, dom_uuid=dom_uuid)
	```

	kvm.py
	```python
	# 封装libvirt connect
	def libvirt_conenct(callback, host, dom_uuid=None):
		callback(...)
	```

	!!! note ""
		这个例子也是阻塞式回调，延迟式回调还没研究过~

## **回调函数类型**

---

有两种类型的回调函数：

1. 阻塞式回调（也叫同步回调）

	> blocking callbacks (also known as synchronous callbacks or just callbacks)

2. 延迟式回调（也叫异步回调）

	> deferred callbacks (also known as asynchronous callbacks)

本页所有例子都是阻塞式回调
