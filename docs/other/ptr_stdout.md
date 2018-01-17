## **环境说明**

---

2台测试vm，1台是单核，1台是4核

代码逻辑:

- 多进程：主进程和2个子进程(fork)同时输出字符串到屏幕

- 多线程：主线程(main)和生成的2个线程同时输出字符串到屏幕

- 协程：主协程和生产的2个协程同时输出字符串到屏幕

观察点:

- 是否有出现串行（注意需要关闭stdout的buffer，否则会影响测试结果），单核应该不会串行，多核应该会。如果出现串行，说明今后使用该语言该版本该模块需要注意锁

结果比较:

- 多进程、多线程、协程，3者之间的区别

- 不同语言、同语言不同版本之间的区别

测试代码:

- 多进程

	??? note "linux-GCC 4.8.5 多进程(fork)"
		```c
		#include <stdio.h>
		#include <unistd.h>

		say(char str[]) {
		    int i;
		    for (i=0; i<20; i++) {
		        sleep(1);
		        printf("%s%d\n", str, i);
		    }
		}

		main() {
		    setbuf(stdout, NULL);
		    pid_t child1_pid=-1;
		    pid_t child2_pid=-1;

		    child1_pid = fork();
		    if (child1_pid > 0) {
		        child2_pid = fork();
		    }

		    if (child1_pid * child2_pid != 0) {
		        say("First");
		    }
		    else if (child1_pid == 0) {
		        say("Second");
		    }
		    else if (child2_pid == 0) {
		        say("Third");
		    }
		    else {
		        printf("ERROR\n");
		    }
		}
		```

	??? note "python2.7 多进程"
		```python
		# -*- coding: utf-8 -*-

		import multiprocessing
		import time

		def say(s):
		    for i in range(20):
		        time.sleep(1)
		        print(s+str(i))

		multiprocessing.Process(target=say, args=("Second",)).start()
		multiprocessing.Process(target=say, args=("Third",)).start()
		say("First")
		```

		python -u xx.py，-u参数表示unbuffered

	??? note "python3.6 多进程"
		```python3
		# -*- coding: utf-8 -*-

		import multiprocessing
		import time

		def say(s):
		    for i in range(20):
		        time.sleep(1)
		        print(s+str(i))

		multiprocessing.Process(target=say, args=("Second",)).start()
		multiprocessing.Process(target=say, args=("Third",)).start()
		say("First")
		```

		python3 -u xx.py，-u参数表示unbuffered

- 多线程

	??? note "linux-GCC 4.8.5 多线程(pthread)"
		```c
		#include <pthread.h>
		#include <stdio.h>

		say(char str[]) {
		    int i;
		    for (i=0; i<20; i++) {
		        sleep(1);
		        printf("%s%d\n", str, i);
		    }
		}

		main() {
		    setbuf(stdout, NULL);
		    pthread_t id;
		    pthread_create(&id,NULL,(void *) say,"Second");
		    pthread_create(&id,NULL,(void *) say,"Third");
		    say("First");
		}
		```

		gcc -l pthread -o thread thread.c

	??? note "python2.7 多线程（thread模块）"
		```python
		# -*- coding: utf-8 -*-

		import thread
		import time

		def say(s):
		    for i in range(20):
		        time.sleep(1)
		        print(s+str(i))

		thread.start_new_thread(say, ("Second",))
		thread.start_new_thread(say, ("Third",))
		say("First")
		```

		python -u xx.py，-u参数表示unbuffered

	??? note "python2.7 多线程（threading模块）"
		```python
		# -*- coding: utf-8 -*-

		import threading
		import time

		def say(s):
		    for i in range(20):
		        time.sleep(1)
		        print(s+str(i))

		threading.Thread(target=say, args=("Second",)).start()
		threading.Thread(target=say, args=("Third",)).start()
		say("First")
		```

		python -u xx.py，-u参数表示unbuffered

	??? note "python3.6 多线程"
		```python3
		# -*- coding: utf-8 -*-

		import threading
		import time

		def say(s):
		    for i in range(20):
		        time.sleep(1)
		        print(s+str(i))

		threading.Thread(target=say, args=("Second",)).start()
		threading.Thread(target=say, args=("Third",)).start()
		say("First")
		```

		python3 -u xx.py，-u参数表示unbuffered

	??? note "java1.8 多线程"
		```java
		public class thread implements Runnable{
		    private thread trt;
		    private String text;

		    public thread(String s) {
		        trt=this;
		        text=s;
		        Thread t=new Thread(trt);
		        t.start();
		    }

		    public static void say(String s) {
		        for(int i=0; i<20; i=i+1) {
		            try {
		                Thread.sleep(1000);
		            } catch (InterruptedException e) {
		                e.printStackTrace();
		            }
		            System.err.println(s+i);
		        }
		    }

		    @Override
		    public void run() {
		        say(text);
		    }

		    public static void main(String[] args) {
		        new thread("Second");
		        new thread("Third");
		        say("First");
		    }
		}
		```

		System.out是有buffer的（暂时没找到关闭buffer方法），而System.err没有buffer

- 协程

	??? note "go1.8 协程"
		```go
		package main

		import (
		    "os"
		    "time"
		    "strconv"
		)

		func say(str string) {
		    for i:=0; i<20; i++ {
		        time.Sleep(1000 * time.Millisecond)
		        os.Stdout.Write([]byte(str + strconv.Itoa(i) + "\n"))
		    }
		}

		func main() {
		    go say("Second")
		    go say("Third")
		    say("First")
		}
		```

		> In C stdout is buffered by default, stderr is not.

		> In Go neither os.Stdout nor os.Stderr is buffered.

## **单核结果**

---

- 多进程

	??? note "linux-GCC 4.8.5 多进程(fork)"
		- 未出现串行
		- 3个进程执行的前后顺序偶尔变化

	??? note "python2.7 多进程"
		- 未出现串行
		- 3个进程执行的前后顺序偶尔变化

	??? note "python3.6 多进程"
		- 未出现串行
		- 3个进程执行的前后顺序偶尔变化

- 多线程

	??? note "linux-GCC 4.8.5 多线程(pthread)"
		```text
		```

	??? note "python2.7 多线程（thread模块）"
		```text
		```

	??? note "python2.7 多线程（threading模块）"
		```text
		```

	??? note "python3.6 多线程"
		```text
		```

	??? note "java1.8 多线程"
		```text
		```

- 协程

	??? note "go1.8 协程"
		```text
		```
