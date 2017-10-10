## **简介**

---

用于放置一个go程序员的所有go代码和依赖在一个workspace里。

!!! note ""
	前提是生成出的二进制文件不重名，如果存在重名情况，就要分成不同的workspace）

## **workspace、repository、package**

---

```text
workspace：包含多个版本控制的repository

	repository：包含多个package

		package：包含多个go源码文件
```

## **标准的workspace本地目录结构**

---

src：go源码文件

pkg：package object（编译出的二进制文件）

bin：可执行文件（编译出的二进制文件）
