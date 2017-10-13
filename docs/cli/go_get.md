## **作用**

---

下载依赖包

为了描述清楚，此处做几个名词定义（我自己定义的）：

当前目录：准备`go install`生成二进制文件的package目录，此处为`$GOPATH/src/github.com/cyent/repo2/myhello`（里面有个`myhello.go`文件`import "github.com/cyent/repo1/stringutil"`）

依赖包：需要用到的包，如`github.com/cyent/repo1/stringutil`

## **实验**

---

1. 在当前目录下执行`go get`：

	下载`github.com/cyent/repo1`到`$GOPATH/src/github.com/cyent/`

	!!! warning ""
		注意这里是下载完整的git仓库，即`github.com/cyent/repo1`仓库，如果`myhello.go`里是import下面路径，比如`github.com/cyent/repo1/stringutil/aaa/aaa.go`，也是下载完整的git仓库。因为git仓库没有办法获得其中某个文件，只能一整个仓库载下来。暂时不知道能否支持svn、http目录、ftp等

	`go install github.com/cyent/repo1/stringutil`

	`go install github.com/cyent/repo2/myhello`

2. 在当前目录下执行`go get github.com/cyent/repo1/stringutil`：

	下载`github.com/cyent/repo1`到`$GOPATH/src/github.com/cyent/`

	`go install github.com/cyent/repo1/stringutil`

3. 在任意路径下执行`go get github.com/cyent/repo1/stringutil`：

	下载`github.com/cyent/repo1`到`$GOPATH/src/github.com/cyent/`

	`go install github.com/cyent/repo1/stringutil`

4. 在任意路径下执行`go get github.com/cyent/repo2/myhello`（假设`github.com/cyent/repo2`本地目录已经存在）：

	下载`github.com/cyent/repo1`到`$GOPATH/src/github.com/cyent/`

	`go install github.com/cyent/repo1/stringutil`

	`go install github.com/cyent/repo2/myhello`

5. 在任意路径下执行`go get github.com/cyent/repo2/myhello`（假设`github.com/cyent/repo2`本地目录不存在）：

	下载`github.com/cyent/repo2`到`$GOPATH/src/github.com/cyent`

	下载`github.com/cyent/repo1`到`$GOPATH/src/github.com/cyent`

	`go install github.com/cyent/repo1/stringutil`

	`go install github.com/cyent/repo2/myhello`

## **实验结论**

---

go get：

1. 判断指定路径（这里的路径指go get /path/to）在本地是否存在：
如果存在，就读取import，然后下载依赖包
如果不存在，就从远程下载包，然后再读取下载的包里的import来判断是否迭代下载依赖包。

2. 依赖包都下载完毕后判断go get是否有带-d参数，如果没有-d，则从迭代的最后一个依赖包开始往前一个个install，直到指定路径（这里的路径指go get /path/to）install完毕

go get参数：

| 参数 | 说明 |
| :-- | :-- |
| -d | 只下载依赖包，不安装。注意，虽然不安装，但是依然会检测目录下是否有可编译文件，如果没有会报错no buildable Go source files in ...（并不影响下载，也不会安装）|
| -f | 强制不检查每个包，适用于local fork of the original，具体使用不了解 |
| -fix | 尚未研究 |
| -insecure | 用于http等不安全的远程地址 |
| -t | 下载和测试 |
| -u | 重新下载依赖包，即使本地workspace已经存在。因为默认情况下如果本地已经存在了就不会再下载了 |
| -v | verbose |

## **最佳实践**

---

如果只想下载依赖包，不想做install，则：go get -d -v /path/to
下载依赖包，并对依赖包做install：go get -v /path/to
（如果不加/path/to，则需要先cd进入当前目录，当然如果当前目录在本地上就没有存在就还是要加/path/to）

下载成功后再手动install当前包：go install -v（如果原来已经存在bin/二进制文件，会自动判断和原来的是否一致，如果一致不会自动编译，如果有更改，会重新编译。即对这个命令不用操心）

如果网络不好，比如需要翻墙才能下载，则只能人工去下载，然后放到指定路径下了
