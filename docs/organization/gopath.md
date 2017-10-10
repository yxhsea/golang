## **简介**

---

GOPATH：workspace的路径，默认为$HOME/go。

!!! warning ""
	GOPATH不是指go安装路径，因此不能写/usr/local/go-1.8之类的

## **如何查看GOPATH**

---

go env可以看到所有go环境变量，如果只看GOPATH可以go env GOPATH

## **GOPATH的设置**

---

执行命令

```text
export GOPATH=/path/to
```

为便于在命令行下执行编译出的二进制可执行程序，建议将bin目录路径加到系统PATH上:

```text
export PATH=$PATH:$(go env GOPATH)/bin
```

另外，可以将上面2个export写入~/.bash_profile，然后source ~/.bash_profile 立即生效
