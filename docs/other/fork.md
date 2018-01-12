创建子进程：

```c
fork();
```

"fork"翻译成中文叫：分叉，表示当执行了一次`fork()`后，会将当前内存数据拷贝一份出来，后续代码执行时候分叉了，即在父进程和子进程里都会执行。

因此在后续代码中如何判断当前的分支是父进程还是哪个子进程就显得尤为重要，尤其是当父进程和子进程需要执行不同后续代码的时候。

!!! note ""
	子进程里再创建子进程，本文暂不做探讨。

## **如何使用fork()创建多个同级的子进程**

---

??? note "创建1个子进程"
	```c
	fork();
	```

??? success "创建2个子进程"
	```c
	pid_t child1_pid;
	pid_t child2_pid;

	child1_pid = fork();
	if (child1_pid > 0) {
		child2_pid = fork();
	}
	```

	可以简写为

	```c
	if (fork() > 0) {
	    fork()
	}
	```

??? note "创建3个子进程"
	```c
	pid_t child1_pid;
	pid_t child2_pid;
	pid_t child3_pid;

	child1_pid = fork();
	if (child1_pid > 0) {
		child2_pid = fork();
	}
	if (child1_pid > 0 && child2_pid > 0) {
		child3_pid = fork();
	}
	```

??? success "创建4个子进程"
	```c
	pid_t child1_pid;
	pid_t child2_pid;
	pid_t child3_pid;
	pid_t child4_pid;

	child1_pid = fork();
	if (child1_pid > 0) {
		child2_pid = fork();
	}
	if (child1_pid > 0 && child2_pid > 0) {
		child3_pid = fork();
	}
	if (child1_pid > 0 && child2_pid > 0 && child3_pid > 0)  {
		child4_pid = fork();
	}
	```

## **如何定位到指定的同级子进程或者父进程**

---

1. 执行`fork();`会返回该子进程的pid（pid_t类型的整型）

2. 当"我是子进程"的时候，我的pid为0

3. 当"我是父进程"的时候，我之前产生的所有子进程pid均大于0

因此可以通过上面3个特点来综合判断"我是谁"，详见[本章节-完整例子](/other/fork/#_4)

## **单核和多核的区别**

---

分别在单核和多核上执行，观察每次执行输出的顺序是否一致，详见[本章节-完整例子](/other/fork/#_4)

## **父进程先退出，子进程会消失吗**

---

在代码最后加上`sleep(1);`来观察（同时可以结合单核和多核分别测试），详见[本章节-完整例子](/other/fork/#_4)

## **完整例子**

---

实验环境：分别在1核和4核linux服务器上测试C语言的fork

??? note "完整代码"
	```c
	#include <stdio.h>
	#include <unistd.h>

	main() {
	    // 主要是为了
	    pid_t child1_pid;
	    pid_t child2_pid;
	    pid_t child3_pid;
	    pid_t child4_pid;

	    // 创建4个同级的子进程
	    child1_pid = fork();
	    if (child1_pid > 0) {
	        child2_pid = fork();
	    }
	    if (child1_pid > 0 && child2_pid > 0) {
	        child3_pid = fork();
	    }
	    if (child1_pid > 0 && child2_pid > 0 && child3_pid > 0)  {
	        child4_pid = fork();
	    }

	    // 定位到指定的子进程或者父进程
	    if (child1_pid * child2_pid * child3_pid * child4_pid != 0) {
	        printf("I'm parent: pid=%d ppid=%d child1_pid=%d child2_pid=%d child3_pid=%d child4_pid=%d\n", getpid(), getppid(), child1_pid, child2_pid, child3_pid, child4_pid);
	    }
	    else if (child1_pid == 0) {
	        printf("I'm child1: pid=%d ppid=%d child1_pid=%d child2_pid=%d child3_pid=%d child4_pid=%d\n", getpid(), getppid(), child1_pid, child2_pid, child3_pid, child4_pid);
	    }
	    else if (child2_pid == 0) {
	        printf("I'm child2: pid=%d ppid=%d child1_pid=%d child2_pid=%d child3_pid=%d child4_pid=%d\n", getpid(), getppid(), child1_pid, child2_pid, child3_pid, child4_pid);
	    }
	    else if (child3_pid == 0) {
	        printf("I'm child3: pid=%d ppid=%d child1_pid=%d child2_pid=%d child3_pid=%d child4_pid=%d\n", getpid(), getppid(), child1_pid, child2_pid, child3_pid, child4_pid);
	    }
	    else if (child4_pid == 0) {
	        printf("I'm child4: pid=%d ppid=%d child1_pid=%d child2_pid=%d child3_pid=%d child4_pid=%d\n", getpid(), getppid(), child1_pid, child2_pid, child3_pid, child4_pid);
	    }
	    else {
	        printf("ERROR\n");
	    }

	    sleep(1);
	}
	```

分别在单核和多核情况下执行上面代码（`gcc -o fork fork.c && ./fork`），同时观察有无`sleep(1)`这行的区别，对比结果：

单核运行结果：

??? note "有`sleep(1)`"
	```text
	I'm parent: pid=4801 ppid=3226 child1_pid=4802 child2_pid=4803 child3_pid=4804 child4_pid=4805
	I'm child4: pid=4805 ppid=4801 child1_pid=4802 child2_pid=4803 child3_pid=4804 child4_pid=0
	I'm child3: pid=4804 ppid=4801 child1_pid=4802 child2_pid=4803 child3_pid=0 child4_pid=-1
	I'm child2: pid=4803 ppid=4801 child1_pid=4802 child2_pid=0 child3_pid=-1 child4_pid=-1
	I'm child1: pid=4802 ppid=4801 child1_pid=0 child2_pid=-1 child3_pid=-1 child4_pid=-1
	```

	多次测试，会发现这5行的前后顺序固定

??? success "无`sleep(1)`"
	```text
	I'm parent: pid=5095 ppid=3226 child1_pid=5096 child2_pid=5097 child3_pid=5098 child4_pid=5099
	I'm child4: pid=5099 ppid=1 child1_pid=5096 child2_pid=5097 child3_pid=5098 child4_pid=0
	I'm child3: pid=5098 ppid=1 child1_pid=5096 child2_pid=5097 child3_pid=0 child4_pid=-1
	I'm child2: pid=5097 ppid=1 child1_pid=5096 child2_pid=0 child3_pid=-1 child4_pid=-1
	I'm child1: pid=5096 ppid=1 child1_pid=0 child2_pid=-1 child3_pid=-1 child4_pid=-1
	```

	多次测试，会发现这5行的前后顺序固定

	注意看上面的ppid，所有child的ppid均为1，是因为父进程一执行完就退出了，而此时子进程还未执行完，随即子进程被过继给操作系统管理（init进程成为继父，pid为1）

多核运行结果：

??? note "有`sleep(1)`"
	```text
	I'm child1: pid=10586 ppid=10585 child1_pid=0 child2_pid=-1 child3_pid=-1 child4_pid=-1
	I'm parent: pid=10585 ppid=3316 child1_pid=10586 child2_pid=10587 child3_pid=10588 child4_pid=10589
	I'm child4: pid=10589 ppid=10585 child1_pid=10586 child2_pid=10587 child3_pid=10588 child4_pid=0
	I'm child3: pid=10588 ppid=10585 child1_pid=10586 child2_pid=10587 child3_pid=0 child4_pid=-1
	I'm child2: pid=10587 ppid=10585 child1_pid=10586 child2_pid=0 child3_pid=-1 child4_pid=-1
	```

	多次测试，会发现这5行的前后顺序不定

??? success "无`sleep(1)`"
	```text
	I'm child1: pid=10787 ppid=10786 child1_pid=0 child2_pid=-1 child3_pid=-1 child4_pid=-1
	I'm child2: pid=10788 ppid=10786 child1_pid=10787 child2_pid=0 child3_pid=-1 child4_pid=-1
	I'm parent: pid=10786 ppid=3316 child1_pid=10787 child2_pid=10788 child3_pid=10789 child4_pid=10790
	I'm child4: pid=10790 ppid=10786 child1_pid=10787 child2_pid=10788 child3_pid=10789 child4_pid=0
	I'm child3: pid=10789 ppid=10786 child1_pid=10787 child2_pid=10788 child3_pid=0 child4_pid=-1
	```

	多次测试，会发现这5行的前后顺序不定

	有时候还会像下面这样结果，即多个ppid为1

	```text
	I'm child1: pid=11443 ppid=11442 child1_pid=0 child2_pid=-1 child3_pid=-1 child4_pid=-1
	I'm child2: pid=11444 ppid=11442 child1_pid=11443 child2_pid=0 child3_pid=-1 child4_pid=-1
	I'm parent: pid=11442 ppid=3316 child1_pid=11443 child2_pid=11444 child3_pid=11445 child4_pid=11446
	I'm child3: pid=11445 ppid=1 child1_pid=11443 child2_pid=11444 child3_pid=0 child4_pid=-1
	I'm child4: pid=11446 ppid=1 child1_pid=11443 child2_pid=11444 child3_pid=11445 child4_pid=0
	```

在代码运行中`pstree -p`的输出如下

```text
fork(16242)─┬─fork(16243)
            ├─fork(16244)
            ├─fork(16245)
            └─fork(16246)
```

注意观察输出中的child{1-4}.pid，可以看到

- 父进程里，4个child的pid都是有值的

- 子进程child1里，child1_pid为0，其他child_pid都为初始值-1

- 子进程child2里，child1_pid有值，child2_pid为0，其他child_pid都为初始值-1

- 子进程child3里，child1_pid有值，child2_pid有值，child3_pid为0，其他child_pid都为初始值-1

- 子进程child4里，child1_pid有值，child2_pid有值，child3_pid有值，child4_pid为0

如果创建同级更多子进程，可以以此类推。通过这个特性来确定当前处于父进程还是某个子进程
