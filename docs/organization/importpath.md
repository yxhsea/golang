## **简介**

---

一个import path（包路径）唯一标识一个package，一个package的import path相当于他所处于workspace中的相对路径。

导入路径可以是本地也可以是在远程，比如开源代码通常放在github.com。当编译时候会自动将远程文件载到本地目录上进行读取

## **组成**

---

import path由base path和package path组成

base path指repository路径，比如github.com/cyent/golang/example（对应本地workspace中的路径$GOPATH/src/github.com/cyent/golang/example）

package path指具体包的路径，比如有个hello包，那么其import path就是github.com/cyent/golang/example/hello

!!! note "github.com也有类似workspace的概念，叫做Organizations"
	比如在github上注册了一个账户叫做cyent，那么github默认会给一个github.com/cyent，之后创建的repo都可以放在这下面，比如github.com/cyent/repo1。

	通常个人用户会像上面这么用，但是对于团队或者公司来说，cyent只是一个注册账号，不作为品牌推广，因此会用团队名，公司名，或产品系列名等，比如github.com/mycompany，这就是建立了一个organization，organization名字叫做mycompany，之后repo都放在这下面，比如github.com/mycompany/myproduct

## **例子**

---

举例说明：假设GOPATH=/opt/cyent/golang，那么本地目录结构:

```text
/opt/cyent/golang/src/
	github.com/organization1/repo
	github.com/organization2/repo
	github.com/cyent/repo1
```

上面这种目录结构解读：
/opt/cyent/golang是放置我的所有go代码的根路径

其中我自己的代码在github.com/cyent/repo1下

organization1/repo和organization2/repo是我代码中会依赖的库（编译时候会自动载到本地来）
