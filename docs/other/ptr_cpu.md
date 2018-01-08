## **环境说明**

---

2台测试vm，1台是单核，1台是4核

多进程代码逻辑：主进程和2个子进程(fork)同时运行死循环

多线程代码逻辑：主线程(main)和生成的2个线程同时运行死循环

观察方法: top(展开所有cpu)和ps -eLf

测试代码:

- 多进程

	??? note "linux-GCC 4.8.5 多进程(fork)"
		```c
		main() {
			if (fork() != 0) {
				fork();
			}
			while(1) {}
		}
		```

	??? note "python2.7 多进程"
		```python
		# -*- coding: utf-8 -*-

		import multiprocessing

		def loop():
			while True:
				pass

		multiprocessing.Process(target=loop).start()
		multiprocessing.Process(target=loop).start()
		loop()
		```

	??? note "python3.6 多进程"
		```python3
		# -*- coding: utf-8 -*-

		import multiprocessing

		def loop():
			while True:
				pass

		multiprocessing.Process(target=loop).start()
		multiprocessing.Process(target=loop).start()
		loop()
		```

- 多线程

	??? note "linux-GCC 4.8.5 多线程(pthread)"
		```c
		#include <pthread.h>

		void loop(void) {
			while(1) {}
		}

		main(void) {
			pthread_t id;
			pthread_create(&id,NULL,(void *) loop,NULL);
			pthread_create(&id,NULL,(void *) loop,NULL);
			loop();
		}
		```

		编译: gcc -l pthread -o thread thread.c

	??? note "python2.7 多线程（thread模块和threading模块效果相同）"
		```python
		# -*- coding: utf-8 -*-

		import thread

		def loop():
			while True:
				pass

		thread.start_new_thread(loop, ())
		thread.start_new_thread(loop, ())
		loop()
		```

	??? note "python3.6 多线程"
		```python3
		# -*- coding: utf-8 -*-

		import threading

		def loop():
			while True:
				pass

		threading.Thread(target=loop).start()
		threading.Thread(target=loop).start()
		loop()
		```

	??? note "java1.8 多线程"
		```java
		public class thread implements Runnable{
			private thread trt;

			public thread() {
				trt=this;
				Thread t1=new Thread(trt);
				Thread t2=new Thread(trt);
				t1.start();
				t2.start();
			}

			@Override
			public void run() {
				while(true) {}
			}

			public static void main(String[] args) {
				new thread();
				while(true) {}
			}
		}
		```



- 协程

	??? note "go1.8 协程"
		```go
		package main

		func loop() {
			for {
			}
		}

		func main() {
			go loop()
			go loop()
			loop()
		}
		```

## **单核**

---

- 多进程

	??? note "linux-GCC 4.8.5 多进程(fork)"

		- top

		```text
		%Cpu0  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		```

		- ps -eLf

		```text
		root    19026 18816 19026 33    1 23:12 pts/0    00:00:09 ./proc
		root    19027 19026 19027 33    1 23:12 pts/0    00:00:09 ./proc
		root    19028 19026 19028 33    1 23:12 pts/0    00:00:09 ./proc
		```

	??? note "python2.7 多进程"

		- top

		```text
		%Cpu0  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		```

		- ps -eLf

		```text
		root    10130  9887 10130 33    1 17:07 pts/1    00:00:09 python pyproc.py
		root    10131 10130 10131 33    1 17:07 pts/1    00:00:09 python pyproc.py
		root    10132 10130 10132 33    1 17:07 pts/1    00:00:09 python pyproc.py
		```

	??? note "python3.6 多进程"

		- top

		```text
		%Cpu0  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		```

		- ps -eLf

		```text
		root    29326 28841 29326 33    1 11:56 pts/1    00:00:06 /usr/local/python-3.6.4/bin/python3 pyproc.py
		root    29327 29326 29327 33    1 11:56 pts/1    00:00:06 /usr/local/python-3.6.4/bin/python3 pyproc.py
		root    29328 29326 29328 33    1 11:56 pts/1    00:00:06 /usr/local/python-3.6.4/bin/python3 pyproc.py
		```

- 多线程

	??? note "linux-GCC 4.8.5 多线程(pthread)"

		- top

		```text
		%Cpu0  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		```

		- ps -eLf

		```text
		root    20956 19390 20956 33    3 23:46 pts/0    00:00:06 ./thread
		root    20956 19390 20957 33    3 23:46 pts/0    00:00:06 ./thread
		root    20956 19390 20958 33    3 23:46 pts/0    00:00:06 ./thread
		```

	??? note "python2.7 多线程（thread模块和threading模块效果相同）"

		- top

		```text
		%Cpu0  : 92.2 us,  7.8 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		```

		- ps -eLf

		```text
		root      9990  9887  9990 33    3 17:06 pts/1    00:00:08 python pythread.py
		root      9990  9887  9991 32    3 17:06 pts/1    00:00:08 python pythread.py
		root      9990  9887  9992 32    3 17:06 pts/1    00:00:08 python pythread.py
		```

	??? note "python3.6 多线程"

		- top

		```text
		%Cpu0  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		```

		- ps -eLf

		```text
		root    29113 28841 29113 33    3 11:55 pts/1    00:00:05 /usr/local/python-3.6.4/bin/python3 pythreading-3.6.py
		root    29113 28841 29114 33    3 11:55 pts/1    00:00:05 /usr/local/python-3.6.4/bin/python3 pythreading-3.6.py
		root    29113 28841 29115 33    3 11:55 pts/1    00:00:05 /usr/local/python-3.6.4/bin/python3 pythreading-3.6.py
		```

	??? note "java1.8 多线程"

		- top

		```text
		%Cpu0  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		```

		- ps -eLf

		```text
		root    16211 16066 16211  0  12 21:39 pts/2    00:00:00 java thread
		root    16211 16066 16212 33  12 21:39 pts/2    00:00:10 java thread
		root    16211 16066 16213  0  12 21:39 pts/2    00:00:00 java thread
		root    16211 16066 16214  0  12 21:39 pts/2    00:00:00 java thread
		root    16211 16066 16215  0  12 21:39 pts/2    00:00:00 java thread
		root    16211 16066 16216  0  12 21:39 pts/2    00:00:00 java thread
		root    16211 16066 16217  0  12 21:39 pts/2    00:00:00 java thread
		root    16211 16066 16218  0  12 21:39 pts/2    00:00:00 java thread
		root    16211 16066 16219  0  12 21:39 pts/2    00:00:00 java thread
		root    16211 16066 16220  0  12 21:39 pts/2    00:00:00 java thread
		root    16211 16066 16221 33  12 21:39 pts/2    00:00:10 java thread
		root    16211 16066 16222 33  12 21:39 pts/2    00:00:10 java thread
		```

- 协程

	??? note "go1.8 协程"

		- top

		```text
		%Cpu0  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		```

		- ps -eLf

		```text
		root    10247  9887 10247 96    3 17:08 pts/1    00:00:19 ./goroutines
		root    10247  9887 10248  0    3 17:08 pts/1    00:00:00 ./goroutines
		root    10247  9887 10249  0    3 17:08 pts/1    00:00:00 ./goroutines
		```


## **多核**

---

- 多进程

	??? note "linux-GCC 4.8.5 多进程(fork)"

		- top

		```text
		%Cpu0  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu1  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu2  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu3  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		```

		- ps -eLf

		```text
		root    20627 17419 20627 99    1 23:10 pts/2    00:00:33 ./proc
		root    20628 20627 20628 99    1 23:10 pts/2    00:00:33 ./proc
		root    20629 20627 20629 99    1 23:10 pts/2    00:00:33 ./proc
		```

	??? note "python2.7 多进程"

		- top

		```text
		%Cpu0  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu1  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu2  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu3  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		```

		- ps -eLf

		```text
		root      4432  2915  4432 99    1 17:31 pts/0    00:00:32 python pyproc.py
		root      4433  4432  4433 99    1 17:31 pts/0    00:00:32 python pyproc.py
		root      4434  4432  4434 99    1 17:31 pts/0    00:00:32 python pyproc.py
		```

	??? note "python3.6 多进程"

		- top

		```text
		%Cpu0  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu1  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu2  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu3  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		```

		- ps -eLf

		```text
		root    16412 16148 16412 99    1 11:57 pts/0    00:00:31 /usr/local/python-3.6.4/bin/python3 pyproc.py
		root    16413 16412 16413 99    1 11:57 pts/0    00:00:31 /usr/local/python-3.6.4/bin/python3 pyproc.py
		root    16414 16412 16414 99    1 11:57 pts/0    00:00:31 /usr/local/python-3.6.4/bin/python3 pyproc.py
		```

- 多线程

	??? note "linux-GCC 4.8.5 多线程(pthread)"

		- top

		```text
		%Cpu0  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu1  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu2  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu3  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		```

		- ps -eLf

		```text
		root    22678 22456 22678 99    3 23:47 pts/0    00:00:57 ./thread
		root    22678 22456 22679 99    3 23:47 pts/0    00:00:57 ./thread
		root    22678 22456 22680 99    3 23:47 pts/0    00:00:57 ./thread
		```

	??? note "python2.7 多线程（thread模块和threading模块效果相同）"

		- top

		```text
		%Cpu0  : 34.3 us, 32.7 sy,  0.0 ni, 31.0 id,  0.0 wa,  0.0 hi,  0.0 si,  2.0 st
		%Cpu1  : 24.6 us, 39.7 sy,  0.0 ni, 34.5 id,  0.0 wa,  0.0 hi,  0.0 si,  1.2 st
		%Cpu2  : 29.2 us, 36.8 sy,  0.0 ni, 33.6 id,  0.0 wa,  0.0 hi,  0.0 si,  0.4 st
		%Cpu3  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		```

		- ps -eLf

		```text
		root      3994  2915  3994 50    3 17:29 pts/0    00:00:27 python pythread.py
		root      3994  2915  3995 47    3 17:29 pts/0    00:00:26 python pythread.py
		root      3994  2915  3996 50    3 17:29 pts/0    00:00:27 python pythread.py
		```

		该程序占用的总cpu保持在150%

	??? note "python3.6 多线程"

		- top

		```text
		%Cpu0  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu1  : 29.7 us,  0.3 sy,  0.0 ni, 70.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu2  : 32.6 us,  0.3 sy,  0.0 ni, 67.1 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu3  : 36.7 us,  0.0 sy,  0.0 ni, 63.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		```

		- ps -eLf

		```text
		root    16643 16148 16643 33    3 11:58 pts/0    00:00:13 /usr/local/python-3.6.4/bin/python3 pythreading.py
		root    16643 16148 16644 32    3 11:58 pts/0    00:00:13 /usr/local/python-3.6.4/bin/python3 pythreading.py
		root    16643 16148 16645 34    3 11:58 pts/0    00:00:13 /usr/local/python-3.6.4/bin/python3 pythreading.py
		```

	??? note "java1.8 多线程"

		- top

		```text
		%Cpu0  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu1  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu2  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu3  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		```

		- ps -eLf

		```text
		root    14651 14429 14651  0  17 21:43 pts/0    00:00:00 java thread
		root    14651 14429 14652 99  17 21:43 pts/0    00:00:59 java thread
		root    14651 14429 14653  0  17 21:43 pts/0    00:00:00 java thread
		root    14651 14429 14654  0  17 21:43 pts/0    00:00:00 java thread
		root    14651 14429 14655  0  17 21:43 pts/0    00:00:00 java thread
		root    14651 14429 14656  0  17 21:43 pts/0    00:00:00 java thread
		root    14651 14429 14657  0  17 21:43 pts/0    00:00:00 java thread
		root    14651 14429 14658  0  17 21:43 pts/0    00:00:00 java thread
		root    14651 14429 14659  0  17 21:43 pts/0    00:00:00 java thread
		root    14651 14429 14660  0  17 21:43 pts/0    00:00:00 java thread
		root    14651 14429 14661  0  17 21:43 pts/0    00:00:00 java thread
		root    14651 14429 14662  0  17 21:43 pts/0    00:00:00 java thread
		root    14651 14429 14663  0  17 21:43 pts/0    00:00:00 java thread
		root    14651 14429 14664  0  17 21:43 pts/0    00:00:00 java thread
		root    14651 14429 14665  0  17 21:43 pts/0    00:00:00 java thread
		root    14651 14429 14666 99  17 21:43 pts/0    00:00:59 java thread
		root    14651 14429 14667 99  17 21:43 pts/0    00:00:59 java thread
		```

	**注意：python2和python3的多线程占用cpu的效果不同，并且和其他语言也都不同**

- 协程

	??? note "go1.8 协程"

		- top

		```text
		%Cpu0  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu1  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu2  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		%Cpu3  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
		```

		- ps -eLf

		```text

		root      5090  2915  5090 99    5 17:33 pts/0    00:00:16 ./goroutines
		root      5090  2915  5091  0    5 17:33 pts/0    00:00:00 ./goroutines
		root      5090  2915  5092  0    5 17:33 pts/0    00:00:00 ./goroutines
		root      5090  2915  5093 99    5 17:33 pts/0    00:00:16 ./goroutines
		root      5090  2915  5094 99    5 17:33 pts/0    00:00:16 ./goroutines
		```
