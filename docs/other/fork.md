本文想说明的：

1. 如何使用fork()创建多个同级的子进程

2. 如何定位到指定的子进程或者父进程，或者说如何知道当前处于哪个进程

3. 通过多进程说明单核和多核的区别

实验环境：分别在单核和多核linux服务器上测试C语言的fork

```c
#include <stdio.h>
#include <unistd.h>

main() {
    // 初始化为-1是为了在能定位到
    pid_t child1_pid=-1;
    pid_t child2_pid=-1;

    child1_pid = fork();
    if (child1_pid != 0) {
        child2_pid = fork();
    }

    if (child1_pid != 0 && child2_pid != 0) {
        printf("I'm parent: pid=%d ppid=%d child1_pid=%d child2_pid=%d\n", getpid(), getppid(), child1_pid, child2_pid);
    }
    else if (child1_pid == 0 && child2_pid != 0) {
        printf("I'm child1: pid=%d ppid=%d child1_pid=%d child2_pid=%d\n", getpid(), getppid(), child1_pid, child2_pid);
    }
    else if (child1_pid != 0 && child2_pid == 0) {
        printf("I'm child2: pid=%d ppid=%d child1_pid=%d child2_pid=%d\n", getpid(), getppid(), child1_pid, child2_pid);
    }
    sleep(1);
}
```

分别在单核和多核情况下执行上面代码，同时把`sleep(1)`这行删掉，对比结果：

单核运行结果：

- 有`sleep(1)`

	```text
	I'm parent: pid=30220 ppid=29448 child1_pid=30221 child2_pid=30222
	I'm child1: pid=30221 ppid=30220 child1_pid=0 child2_pid=-1
	I'm child2: pid=30222 ppid=30220 child1_pid=30221 child2_pid=0
	```

	!!! warning ""
		多次测试，会发现这3行的前后顺序固定

- 无`sleep(1)`

	```text
	I'm parent: pid=22414 ppid=20062 child1_pid=22415 child2_pid=22416
	I'm child2: pid=22416 ppid=1 child1_pid=22415 child2_pid=0
	I'm child1: pid=22415 ppid=1 child1_pid=0 child2_pid=-1
	```

	!!! warning ""
		多次测试，会发现这3行的前后顺序固定

多核运行结果：

- 有`sleep(1)`

	```text
	I'm parent: pid=21585 ppid=20062 child1_pid=21586 child2_pid=21587
	I'm child2: pid=21587 ppid=21585 child1_pid=21586 child2_pid=0
	I'm child1: pid=21586 ppid=21585 child1_pid=0 child2_pid=-1
	```

	!!! warning ""
		多次测试，会发现这3行的前后顺序不定

- 无`sleep(1)`

	```text
	I'm child1: pid=32181 ppid=32180 child1_pid=0 child2_pid=-1
	I'm parent: pid=32180 ppid=29448 child1_pid=32181 child2_pid=32182
	I'm child2: pid=32182 ppid=1 child1_pid=32181 child2_pid=0
	```

	!!! warning ""
		多次测试，会发现这3行的前后顺序不定
