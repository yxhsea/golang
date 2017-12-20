下面是从shell到python再到golang，从面向对象都不懂到了解面向对象、了解interface的整个思考过程。

**注意：本页只适合编程小白看，并且要不然别看，要看就要看到底，因为中间经历了多次的错误例子和总结，直到最后才抓到面向对象和接口的尾巴**

!!! note ""
	本页大量使用了伪代码，所以不保证例子都是可执行的

创建netns，netns名组成：VRouter-${name}

```shell
create_vrouter() {
    name="$1"
    ip netns add VRouter-${name}
}

create_vrouter "vrtest"
```

这串代码有2处耦合：

1. 创建netns与创建vrouter耦合

2. netns名与创建vrouter耦合

第一个耦合的解决：将netns的创建封装为一个函数

```shell
create_netns() {
    name="$1"
    ip netns add ${name}
}

create_vrouter() {
    name="$1"
    vrname="VRouter-${name}"
    create_netns ${vrname}
}

create_vrouter "vrtest"
```

第二个耦合的解决：将netns封装为一个函数

```shell
get_ns_name() {
    name="$1"
    echo "VRouter-${name}"
}

create_netns() {
    name="$1"
    ns_name=`get_ns_name ${name}`
    ip netns add ${ns_name}
}

create_vrouter() {
    name="$1"
    create_netns ${name}
}

create_vrouter "vrtest"
```

上面这种方法是create_netns函数里调用get_ns_name，下面这种方法是在create_vrouter里调用get_ns_name

```shell
get_ns_name() {
    name="$1"
    echo "VRouter-${name}"
}

create_netns() {
    name="$1"
    ip netns add ${name}
}

create_vrouter() {
    name="$1"
    ns_name=`get_ns_name ${name}`
    create_netns ${ns_name}
}
```

这2种方法看过去好像都行，于是接下来做个扩展，netns名加ext、int，看下会发生什么：

```shell
get_ns_name() {
    name="$1"
    type="$2"
    if [ "${type}" = "ext" ]; then
        echo "E-VRouter-${name}"
    elif [ "${type}" = "int" ]; then
        echo "I-VRouter-${name}"
    else
        echo "VRouter-${name}"
    fi
}

create_netns() {
    name="$1"
    type="$2"
    ns_name=`get_ns_name ${name} ${type}`
    ip netns add ${ns_name}
}

create_vrouter() {
    name="$1"
    type="$2"
    create_netns ${name} ${type}
}

create_vrouter "vrtest" "ext"
```

减少代码重复是本质，但是实现起来有多种方式，以前我都是停留在没有任何思考，能多减少一行是一行的想法来编写代码，导致了各种耦合，以致于代码扩展变得很难。接下来要找到一种比较好的方式。

相似但不相关的东西不要耦合在一起，比如dhcp需要netns、vrouter也需要netns，但创建netns要分成2个函数，分别是dhcp的netns和vrouter的netns，否则后期因为vrouter的netns变化，就有可能连累dhcp的netns（往往到了这个时候为了不连累，又会新搞一个函数叫做dhcp_netns，这样就会造成netns和dhcp_netns同时存在的局面，维护就困难了）。

突然想到，有一种情况，可能需要独立搞个netns函数，就是当这个函数里做的东西有不同环境的类型判断，比如启动host，在libvirt环境是virsh start，而在物理机是ipmitool power on，在docker环境是docker start。这种时候封装一个start_host函数，根据传入参数或其他途径判断类型，怎么启动host。

但问题来了，这种情况的话，岂不是又回到最初问题了，就是什么时候要独立出来成为一个独立函数或独立模块，什么时候不要独立

于是想到答案：当满足2个条件时候独立出来

1. 不和调用者代码逻辑耦合。比如create_netns函数里就接收name，而不要和I-VRouter、E-VRouter这种代码逻辑耦合一起，到底是I-VRouter还是E-VRouter还是I-dhcp，由调用者(主函数)自己来处理

2. 要处理的事情较多时候，比如现在create_netns只有一种方式ip netns add，但也许在不同的操作系统上命令不一样（比如某些系统是用net命令替代ip命令，或者今后操作系统更新后废掉了ip命令，而是直接netns add命令等等）

所以上面的shell应该这么改：

第一版（没有类型）：

```shell
create_netns() {
    name="$1"
    ip netns add ${name}
}

create_vrouter() {
    name="$1"
    create_netns ${name}
}

create_vrouter "vrtest"
```

第二版（增加类型）：

```shell
create_netns() {
    name="$1"
    ip netns add ${name}
}

create_vrouter() {
    name="$1"
    type="$2"
    if [ "${type}" = "ext" ]; then
        ns_name="E-VRouter-${name}"
    elif [ "${type}" = "int" ]; then
        ns_name="I-VRouter-${name}"
    else
        ns_name="VRouter-${name}"
    fi
    create_netns ${ns_name}
}

create_vrouter "vrtest" "ext"
```

如果要把ns_name封装起来也可以，但是只用于vrouter的netns，函数名可以叫做get_vrouter_netns_name()

上面虽然用到了封装，但也只是面向过程的思路，如果用python，则可以使用面向对象：

定义2个类：vrouter类、netns类

```python
class netns:
    def __init__(self, name):
        self.name = name
    def create(self):
        ip.netns.add(self.name)

class Vrouter:
    def __init__(self, name, stype):
        self.name = name
        self.stype = stype
    def create(self):
        if self.stype == "int":
            nsname = "I-VRouter-"+self.name
        elif self.stype == "ext":
            nsname = "E-VRouter-"+self.name
        else:
            nsname = "VRouter-"+self.name
        ns = netns(nsname)
        ns.create()

vr = Vrouter('vabcdefg', 'int')
vr.create()
```

接下来用go，但先不用接口（就2个函数，也没必要用接口）：

```go
package main

import "fmt"

type netns struct {
    name string
}
func (n *netns) create() {
    fmt.Println(n.name)
}

type vrouter struct {
    name string
    stype string
}
func (v *vrouter) create() {
    var nsname string
    switch v.stype {
    case "ext":
        nsname = "E-VRouter-"+v.name
    case "int":
        nsname = "I-VRouter-"+v.name
    default:
        nsname = "VRouter-"+v.name
    }

    ns := &netns{name: nsname}
    ns.create()
}

func main() {
    vr := &vrouter{name: "vrtest", stype: "int"}
    vr.create()
}
```

输出

```text
I-VRouter-vrtest
```

接下来继续增加功能点，增加p2p（veth）及ip：

shell：

```shell
create_netns() {
    name="$1"
    ip netns add ${name}
}

create_veth() {
    name="$1"
    peername="$2"
    ip link add ${name} type veth peer name ${peername}
}

up_nic() {
    name="$1"
    ip link set ${name} up
}

set_nic_ns() {
    nic="$1"
    ns="$2"
    ip link set ${nic} netns ${ns}
}

set_nic_ipv4() {
    nic="$1"
    ipmask="$2"
    ns="$3"
    if [ -n "${ns}" ]; then
        ip netns exec ${ns} ip address add ${ipmask} dev ${nic}
    else
        ip address add ${ipmask} dev ${nic}
    fi
}

set_nic_br() {
    nic="$1"
    br="$2"
    ns="$3"
    if [ -n "${ns}" ]; then
        ip netns exec ${ns} ip link set ${nic} master ${br}
    else
        ip link set ${nic} master ${br}
    fi
}

set_route() {
    route="$1"
    via="$2"
    dev="$3"
    ns="$4"
    if [ -n "${ns}" ]; then
        if [ -n "${dev}" ]; then
            ip netns exec ${ns} ip route add ${route} via ${via} dev ${dev}
        else
            ip netns exec ${ns} ip route add ${route} via ${via}
        fi
    else
        if [ -n "${dev}" ]; then
            ip route add ${route} via ${via} dev ${dev}
        else
            ip route add ${route} via ${via}
        fi
    fi
}

create_vrouter() {
    name="$1"
    type="$2"
    ipmask="$3"
    gw="$4"
    if [ "${type}" = "ext" ]; then
        ns_name="E-VRouter-${name}"
        p2p_name="pie-${name}"
        p2p_peername="poe-${name}"
    elif [ "${type}" = "int" ]; then
        ns_name="I-VRouter-${name}"
        p2p_name="pii-${name}"
        p2p_peername="poi-${name}"
    else
        ns_name="VRouter-${name}"
        p2p_name="pi-${name}"
        p2p_peername="po-${name}"
    fi
    create_netns ${ns_name}
    create_veth ${p2p_name} ${p2p_peername}
    up_nic ${p2p_name}
    up_nic ${p2p_peername}
    set_nic_ns ${p2p_peername} ${ns_name}
    set_nic_ipv4 ${p2p_peername} ${ipmask} ${ns_name}
    set_route "0.0.0.0/0" ${gw} ${p2p_peername} ${ns_name}
    set_nic_br ${p2p_name} "br-int-legacy"
}

create_vrouter "vrtest" "ext" "192.168.100.1/24" "192.168.100.254"
```

python：

```python
class netns:
    def __init__(self):
        pass
    def create(self, name):
        ip.netns.add(name)

class iproute:
    def __init__(self):
        pass
    def create_veth(self, name, peername):
        ip.link.add.name.type.veth.peer.peername
    def up_nic(self, nic):
        ip.link.set.nic.up
    def set_nic_ns(self, nic, ns):
        ip.link.set.nic.netns.ns
    def set_nic_ipv4(self, nic, ipmask, ns=None):
        if ns == None:
            ip.netns.exec.ns.ip.address.add.ipmask.dev.nic
        else:
            ip.address.add.ipmask.dev.nic
    def set_route(self, route, via, dev=None, ns=None):
        if ns == None:
            if dev == None:
                ip.netns.exec.ns.ip.route.add.route.via.via.dev.dev
            else:
                ip.netns.exec.ns.ip.route.add.route.via.via
        else:
            if dev == None:
                ip.route.add.route.via.via.dev.dev
            else:
                ip.route.add.route.via.via
    def set_nic_br(self, nic, br, ns):
        if ns == None:
            ip.netns.exec.ns.ip.link.set.nic.master.br
        else:
            ip.link.set.nic.master.br

class Vrouter:
    def __init__(self, name, stype, ipmask, gw):
        pass
    def create(self, name, stype, ipmask, gw):
        if self.stype == "int":
            nsname = "I-VRouter-"+name
            p2p_name = "pii-"+name
            p2p_peername = "poi-"+name
        elif self.stype == "ext":
            nsname = "E-VRouter-"+name
            p2p_name = "pie-"+name
            p2p_peername = "poe-"+name
        else:
            nsname = "VRouter-"+name
            p2p_name = "pi-"+name
            p2p_peername = "po-"+name

        ns = netns()
        ns.create(nsname)
        ip = iproute()
        ip.create_veth(p2p_name, p2p_peername)
        ip.up_nic(p2p_name)
        ip.up_nic(p2p_peername)
        ip.set_nic_ns(p2p_peername, nsname)
        ip.set_nic_ipv4(p2p_peername, ipmask, nsname)
        ip.set_route("0.0.0.0/0", gw, p2p_peername, nsname)
        ip.set_nic_br(p2p_name, "br-int-legacy")

vr = Vrouter()
vr.create("vrtest", "ext", "192.168.100.1/24", "192.168.100.254")
```

然后，神奇出现了，当我想把上面这段代码直接翻译成golang时候，发现写不下去，因为golang里模拟面向对象通常用的是struct来模拟属性，用签名函数模拟方法，那么问题来了。上面py代码里的iproute类没有设定属性，只设定方法，而go的struct没有设置元素就觉得怪怪的（其实go里struct没有元素也很常见）。

到此，开始意识到，我并没有真正理解面向对象的概念，我将python的class当封装拿来用（比如上面的iproute类），而不是当作真正的面向对象。那么接下来，先对python做改造，做成真正的面向对象，然后再试着用go来写

python：

```python
# _*_ coding: utf-8 _*_

class Netns(object):
    def __init__(self, name):
        self.name = name

    def add(self):
        print "ip netns add {}".format(self.name)

    def delete(self):
        print "ip netns del {}".format(self.name)

class Nic(object):
    def __init__(self, name, netns=None):
        self.name = name        # 网卡名
        self.netns = netns      # 网卡当前所在netns

    def add(self):
        print "Need to override or overload!"

    def delete(self):
        if self.netns == None:
            print "ip link delete {}".format(self.name)
        else:
            print "ip netns exec {} ip link delete {}".format(self.netns, self.name)

    def up(self):
        if self.netns == None:
            print "ip link set {} up".format(self.name)
        else:
            print "ip netns exec {} ip link set {} up".format(self.netns, self.name)

    def down(self):
        if self.netns == None:
            print "ip link set {} down".format(self.name)
        else:
            print "ip netns exec {} ip link set {} down".format(self.netns, self.name)

    def setMaster(self, master):
        if self.netns == None:
            print "ip link set {} master {}".format(self.name, master)
        else:
            print "ip netns exec {} ip link set {} master {}".format(self.netns, self.name, master)

    def setNetns(self, netns):
        '''
        将当前网络命名空间(self.netns)中的网卡放置到新的命名空间中
        '''
        if self.netns == None:
            print "ip link set {} netns {}".format(self.name, netns)
        else:
            print "ip netns exec {} ip link set {} netns {}".format(self.netns, self.name, netns)
        self.netns = netns

    def addIPAddr(self, ip):
        if self.netns == None:
            print "ip address add {} dev {}".format(ip, self.name)
        else:
            print "ip netns exec {} ip address add {} dev {}".format(self.netns, ip, self.name)

    def isExists(self):
        if self.netns == None:
            print "ip link show dev {} >/dev/null 2>&1".format(self.name)
        else:
            print "ip netns exec {} ip link show dev {} >/dev/null 2>&1".format(self.netns, self.name)

class Vtep(Nic):
    def __init__(self, name, vni, dev, port, netns=None, group=None):
        Nic.__init__(self, name, netns)
        self.vni = vni
        self.group = group
        self.dev = dev
        self.port = port

    def add(self):
        if self.group == None:
            if self.netns == None:
                print "ip link add {} mtu 1500 type vxlan id {} ttl 64 dev {} dstport {}".format(
                    self.name,
                    self.vni,
                    self.dev,
                    self.port
                )
            else:
                print "ip netns exec {} ip link add {} mtu 1500 type vxlan id {} ttl 64 dev {} dstport {}".format(
                    self.netns,
                    self.name,
                    self.vni,
                    self.dev,
                    self.port
                )
        else:
            if self.netns == None:
                print "ip link add {} mtu 1500 type vxlan id {} group {} ttl 64 dev {} dstport {}".format(
                    self.name,
                    self.vni,
                    self.group,
                    self.dev,
                    self.port,
                )
            else:
                print "ip netns exec {} ip link add {} mtu 1500 type vxlan id {} group {} ttl 64 dev {} dstport {}".format(
                    self.netns,
                    self.name,
                    self.vni,
                    self.group,
                    self.dev,
                    self.port
                )

class Bridge(Nic):
    def __init__(self, name, netns=None):
        Nic.__init__(self, name, netns)

    def add(self):
        if self.netns == None:
            print "ip link add {} type bridge".format(self.name)
        else:
            print "ip netns exec {} ip link add {} type bridge".format(self.netns, self.name)

class Veth(Nic):
    def __init__(self, name, peer=None, netns=None):
        Nic.__init__(self, name, netns)
        self.peer = peer

    def add(self):
        if self.peer == None:
            if self.netns == None:
                print "ip link add {} type veth peer".format(self.name)
            else:
                print "ip netns exec {} ip link add {} type veth peer".format(self.netns, self.name)
            # 假设获得的peer是vethX
            self.peer = "vethX"
            return self.peer
        else:
            if self.netns == None:
                print "ip link add {} type veth peer name {}".format(self.name, self.peer)
            else:
                print "ip netns exec {} ip link add {} type veth peer name {}".format(self.netns, self.name, self.peer)

print "=====class Netns test====="
ns = Netns(name="ns1")
ns.add()
ns.delete()
print

print "=====class Bridge test====="
print "---no netns---"
br1 = Bridge(name="br1")
br1.isExists()
br1.add()
br1.up()
br1.down()
br1.delete()
print "---netns=ns1---"
br2 = Bridge(name="br1", netns="ns1")
br2.isExists()
br2.add()
br2.up()
br2.down()
br2.delete()
print

print "=====class Vtep test====="
print "---multi vtep---"
print "-no netns-"
mvtep1 = Vtep(
    name="int-v1111111",
    vni=1111111,
    group="239.1.111.111",
    dev="em1",
    port=4789
)
mvtep1.isExists()
mvtep1.add()
mvtep1.setMaster(master="br1")
mvtep1.up()
mvtep1.down()
mvtep1.delete()
print "---multi vtep---"
print "-netns=ns1-"
mvtep2 = Vtep(
    name="int-v1111111",
    vni=1111111,
    group="239.1.111.111",
    dev="em1",
    port=4789,
    netns="ns1"
)
mvtep2.isExists()
mvtep2.add()
mvtep2.setMaster(master="br1")
mvtep2.up()
mvtep2.down()
mvtep2.delete()

print "---uni vtep---"
print "-no netns-"
uvtep1 = Vtep(
    name="int-v100",
    vni=100,
    dev="em1",
    port=4789
)
uvtep1.isExists()
uvtep1.add()
uvtep1.setMaster(master="br1")
uvtep1.up()
uvtep1.down()
uvtep1.delete()
print "-netns=ns1-"
uvtep2 = Vtep(
    name="int-v100",
    vni=100,
    dev="em1",
    port=4789,
    netns="ns1"
)
uvtep2.isExists()
uvtep2.add()
uvtep2.setMaster(master="br1")
uvtep2.up()
uvtep2.down()
uvtep2.delete()
print

print "=====class Veth test====="
print "---no peer---"
print "--no netns--"
veth1a = Veth(name="pii-xxx")
veth1a.add()
veth1a.up()
veth1a.setNetns(netns="ns1")
print

print "--netns=ns1--"
veth2a = Veth(name="pii-xxx", netns="ns1")
veth2a.add()
veth2a.up()
veth2a.setNetns(netns="ns2")
print

print "---peer=poi-xxx---"
print "--no netns--"
veth3a = Veth(name="pii-xxx", peer="poi-xxx")
veth3a.add()
veth3a.up()
veth3a.setNetns(netns="ns1")
print

print "--netns=ns1--"
veth4a = Veth(name="pii-xxx", peer="poi-xxx", netns="ns1")
veth4a.add()
veth4a.up()
veth4a.setNetns(netns="ns2")
```

输出

```text
=====class Netns test=====
ip netns add ns1
ip netns del ns1

=====class Bridge test=====
---no netns---
ip link show dev br1 >/dev/null 2>&1
ip link add br1 type bridge
ip link set br1 up
ip link set br1 down
ip link delete br1
---netns=ns1---
ip netns exec ns1 ip link show dev br1 >/dev/null 2>&1
ip netns exec ns1 ip link add br1 type bridge
ip netns exec ns1 ip link set br1 up
ip netns exec ns1 ip link set br1 down
ip netns exec ns1 ip link delete br1

=====class Vtep test=====
---multi vtep---
-no netns-
ip link show dev int-v1111111 >/dev/null 2>&1
ip link add int-v1111111 mtu 1500 type vxlan id 1111111 group 239.1.111.111 ttl 64 dev em1 dstport 4789
ip link set int-v1111111 master br1
ip link set int-v1111111 up
ip link set int-v1111111 down
ip link delete int-v1111111
---multi vtep---
-netns=ns1-
ip netns exec ns1 ip link show dev int-v1111111 >/dev/null 2>&1
ip netns exec ns1 ip link add int-v1111111 mtu 1500 type vxlan id 1111111 group 239.1.111.111 ttl 64 dev em1 dstport 4789
ip netns exec ns1 ip link set int-v1111111 master br1
ip netns exec ns1 ip link set int-v1111111 up
ip netns exec ns1 ip link set int-v1111111 down
ip netns exec ns1 ip link delete int-v1111111
---uni vtep---
-no netns-
ip link show dev int-v100 >/dev/null 2>&1
ip link add int-v100 mtu 1500 type vxlan id 100 ttl 64 dev em1 dstport 4789
ip link set int-v100 master br1
ip link set int-v100 up
ip link set int-v100 down
ip link delete int-v100
-netns=ns1-
ip netns exec ns1 ip link show dev int-v100 >/dev/null 2>&1
ip netns exec ns1 ip link add int-v100 mtu 1500 type vxlan id 100 ttl 64 dev em1 dstport 4789
ip netns exec ns1 ip link set int-v100 master br1
ip netns exec ns1 ip link set int-v100 up
ip netns exec ns1 ip link set int-v100 down
ip netns exec ns1 ip link delete int-v100

=====class Veth test=====
---no peer---
--no netns--
ip link add pii-xxx type veth peer
ip link set pii-xxx up
ip link set pii-xxx netns ns1

--netns=ns1--
ip netns exec ns1 ip link add pii-xxx type veth peer
ip netns exec ns1 ip link set pii-xxx up
ip netns exec ns1 ip link set pii-xxx netns ns2

---peer=poi-xxx---
--no netns--
ip link add pii-xxx type veth peer name poi-xxx
ip link set pii-xxx up
ip link set pii-xxx netns ns1

--netns=ns1--
ip netns exec ns1 ip link add pii-xxx type veth peer name poi-xxx
ip netns exec ns1 ip link set pii-xxx up
ip netns exec ns1 ip link set pii-xxx netns ns2
```

上面代码用go实现：

建一个iproute目录，有bridge、netns、nic、veth、vtep这几个子目录，存放基础库：

netns/netns.go

```go
package netns

import "fmt"

type netns struct {
    name string
}

func (this *netns) Add() {
    fmt.Printf("ip netns add %s\n", this.name)
}

func (this *netns) Del() {
    fmt.Printf("ip netns del %s\n", this.name)
}

func NewNetns(name string) *netns { return &netns{name: name} }
```

nic/nic.go

```go
package nic

import "fmt"

type Nic struct {
    Name  string
    Netns string
}

func (this *Nic) Add() {
    fmt.Println("Need to override or overload!")
}

func (this *Nic) Del() {
    if this.Netns == "" {
        fmt.Printf("ip link delete %s\n", this.Name)
    } else {
        fmt.Printf("ip netns exec %s ip link delete %s\n",
            this.Netns,
            this.Name,
        )
    }
}

func (this *Nic) Up() {
    if this.Netns == "" {
        fmt.Printf("ip link set %s up\n", this.Name)
    } else {
        fmt.Printf("ip netns exec %s ip link set %s up\n",
            this.Netns,
            this.Name,
        )
    }
}

func (this *Nic) Down() {
    if this.Netns == "" {
        fmt.Printf("ip link set %s down\n", this.Name)
    } else {
        fmt.Printf("ip netns exec %s ip link set %s down\n",
            this.Netns,
            this.Name,
        )
    }
}

func (this *Nic) SetMaster(master string) {
    if this.Netns == "" {
        fmt.Printf("ip link set %s master %s\n",
            this.Name,
            master,
        )
    } else {
        fmt.Printf("ip netns exec %s ip link set %s master %s\n",
            this.Netns,
            this.Name,
            master,
        )
    }
}

func (this *Nic) SetNetns(netns string) {
    if this.Netns == "" {
        fmt.Printf("ip link set %s netns %s\n",
            this.Name,
            netns,
        )
    } else {
        fmt.Printf("ip netns exec %s ip link set %s netns %s\n",
            this.Netns,
            this.Name,
            netns,
        )
    }
    this.Netns = netns
}

func (this *Nic) AddIPAddr(ip string) {
    if this.Netns == "" {
        fmt.Printf("ip address add %s dev %s\n",
            ip,
            this.Name,
        )
    } else {
        fmt.Printf("ip netns exec %s ip address add %s dev %s\n",
            this.Netns,
            ip,
            this.Name,
        )
    }
}

func (this *Nic) IsExists() {
    if this.Netns == "" {
        fmt.Printf("ip link show dev %s >/dev/null 2>&1\n", this.Name)
    } else {
        fmt.Printf("ip netns exec %s ip link show dev %s >/dev/null 2>&1\n", this.Netns, this.Name)
    }
}
```

bridge/bridge.go

```go
package bridge

import (
    "fmt"
    "github.com/cyent/repo1/iproute/nic"
)

type bridge struct {
    nic.Nic
    name  string
    netns string
}

func (this *bridge) Add() {
    if this.netns == "" {
        fmt.Printf("ip link add %s type bridge\n", this.name)
    } else {
        fmt.Printf("ip netns exec %s ip link add %s type bridge\n", this.netns, this.name)
    }
}

func NewBridge(name string, netns string) *bridge {
    b := new(bridge)
    b.Nic.Name = name
    b.Nic.Netns = netns
    b.name = name
    b.netns = netns
    return b
}
```

veth/veth.go

```go
package veth

import (
    "fmt"
    "github.com/cyent/repo1/iproute/nic"
)

type veth struct {
    nic.Nic
    name  string
    peer  string
    netns string
}

func (this *veth) Add() {
    if this.peer == "" {
        if this.netns == "" {
            fmt.Printf("ip link add %s type veth peer\n", this.name)
        } else {
            fmt.Printf("ip netns exec %s ip link add %s type veth peer\n", this.netns, this.name)
        }
        // 假设获得的peer是vethX
        this.peer = "vethX"
    } else {
        if this.netns == "" {
            fmt.Printf("ip link add %s type veth peer name %s\n", this.name, this.peer)
        } else {
            fmt.Printf("ip netns exec %s ip link add %s type veth peer name %s\n", this.netns, this.name, this.peer)
        }
    }
}

func New(name string, peer string, netns string) *veth {
    v := new(veth)
    v.Nic.Name = name
    v.Nic.Netns = netns
    v.name = name
    v.peer = peer
    v.netns = netns
    return v
}
```

vtep/vtep.go

```go
package vtep

import (
    "fmt"
    "github.com/cyent/repo1/iproute/nic"
)

type vtep struct {
    nic.Nic
    name  string
    vni   int
    dev   string
    port  int
    netns string
    group string
}

func (this *vtep) Add() {
    if this.group == "" {
        if this.netns == "" {
            fmt.Printf("ip link add %s mtu 1500 type vxlan id %d ttl 64 dev %s dstport %d\n",
                this.name,
                this.vni,
                this.dev,
                this.port,
            )
        } else {
            fmt.Printf("ip netns exec %s ip link add %s mtu 1500 type vxlan id %d ttl 64 dev %s dstport %d\n",
                this.netns,
                this.name,
                this.vni,
                this.dev,
                this.port,
            )
        }
    } else {
        if this.netns == "" {
            fmt.Printf("ip link add %s mtu 1500 type vxlan id %d group %s ttl 64 dev %s dstport %d\n",
                this.name,
                this.vni,
                this.group,
                this.dev,
                this.port,
            )
        } else {
            fmt.Printf("ip netns exec %s ip link add %s mtu 1500 type vxlan id %d group %s ttl 64 dev %s dstport %d\n",
                this.netns,
                this.name,
                this.vni,
                this.group,
                this.dev,
                this.port,
            )
        }
    }
}

func NewVtep(name string, vni int, dev string, port int, netns string, group string) *vtep {
    v := new(vtep)
    v.Nic.Name = name
    v.Nic.Netns = netns
    v.name = name
    v.netns = netns
    v.vni = vni
    v.dev = dev
    v.port = port
    v.group = group
    return v
}
```

调用者

```go
package main

import (
    "fmt"
    "github.com/cyent/repo1/iproute/bridge"
    "github.com/cyent/repo1/iproute/netns"
    "github.com/cyent/repo1/iproute/veth"
    "github.com/cyent/repo1/iproute/vtep"
)

type vnic interface {
    Add()
    Del()
    Up()
    Down()
    SetMaster(string)
    SetNetns(string)
    AddIPAddr(string)
    IsExists()
}

func main() {
    fmt.Println("=====class Netns test=====")
    ns := netns.New("ns1")
    ns.Add()
    ns.Del()
    fmt.Println()

    var i vnic

    fmt.Println("=====class Bridge test=====")
    fmt.Println("---no netns---")
    i = bridge.New("br1", "")
    i.IsExists()
    i.Add()
    i.Up()
    i.Down()
    i.Del()
    fmt.Println("---netns=ns1---")
    i = bridge.New("br1", "ns1")
    i.IsExists()
    i.Add()
    i.Up()
    i.Down()
    i.Del()
    fmt.Println()

    fmt.Println("=====class Vtep test=====")
    fmt.Println("---multi vtep---")
    fmt.Println("-no netns-")
    i = vtep.New("int-v1111111", 1111111, "em1", 4789, "", "239.1.111.111")
    i.IsExists()
    i.Add()
    i.SetMaster("br1")
    i.Up()
    i.Down()
    i.Del()
    fmt.Println("---multi vtep---")
    fmt.Println("-netns=ns1-")
    i = vtep.New("int-v1111111", 1111111, "em1", 4789, "ns1", "239.1.111.111")
    i.IsExists()
    i.Add()
    i.SetMaster("br1")
    i.Up()
    i.Down()
    i.Del()

    fmt.Println("---uni vtep---")
    fmt.Println("-no netns-")
    i = vtep.New("int-v100", 100, "em1", 4789, "", "")
    i.IsExists()
    i.Add()
    i.SetMaster("br1")
    i.Up()
    i.Down()
    i.Del()
    fmt.Println("-netns=ns1-")
    i = vtep.New("int-v100", 100, "em1", 4789, "ns1", "")
    i.IsExists()
    i.Add()
    i.SetMaster("br1")
    i.Up()
    i.Down()
    i.Del()
    fmt.Println()

    fmt.Println("=====class Veth test=====")
    fmt.Println("---no peer---")
    fmt.Println("--no netns--")
    i = veth.New("pii-xxx", "", "")
    i.Add()
    i.Up()
    i.SetNetns("ns1")
    fmt.Println()

    fmt.Println("--netns=ns1--")
    i = veth.New("pii-xxx", "", "ns1")
    i.Add()
    i.Up()
    i.SetNetns("ns2")
    fmt.Println()

    fmt.Println("---peer=poi-xxx---")
    fmt.Println("--no netns--")
    i = veth.New("pii-xxx", "poi-xxx", "")
    i.Add()
    i.Up()
    i.SetNetns("ns1")
    fmt.Println()

    fmt.Println("--netns=ns1--")
    i = veth.New("pii-xxx", "poi-xxx", "ns1")
    i.Add()
    i.Up()
    i.SetNetns("ns2")
}
```

输出和刚才python的输出内容完全一样

上面这个过程中有一个很有意思的一点，就是interface让我真正看清了事物的本质：

一开始在iproute.py代码里，有这么一段：

在class Veth里有重载了SetNetns方法：

```python
class Veth:
    ...
    def setNetns(self, isPeer=False)
```

即当调用该方法时候如果没有带isPeer参数或者isPeer为False时候，将self.name加入到netns中，而当isPeer=True时候，则将self.peer加入到netns中。

这个逻辑咋一看没有任何问题，然后用go来写（此时还没用interface）：

在veth.go里：

```go
func (this *veth) SetNetns(isPeer bool):
```

咋一看也没有问题，调用该方法时候如果isPeer为false，将this.name加入到netns中，而当isPeer=true时候，则将this.peer加入到netns中。

但有意思的来了，在调用者这里写一个接口：

```go
type vnic interface {
    Add()
    Del()
    Up()
    Down()
    SetMaster(string)
    SetNetns(string)
    AddIPAddr(string)
    IsExists()
}
```

这样就有问题了，因为bridge、vtep的SetNetns没有参数，而veth的SetNetns有参数，这样就造成了veth struct没有实现vnic接口。

然后又想着，那能否vnic接口里把SetNetns给去掉，但这样就会造成：

```go
var i vnic
i = bridge.New(xx)
i.Add()    //不报错
i.Del()    //不报错
i.SetNetns()    //报错
```

额，于是定下心仔细思考了下：

veth虽然属于nic类，但是nic面向的是一张网卡，而不是多张网卡，因此veth创建出来有2张网卡，自身名网卡属于nic，peer网卡只是个附属品。

因此要操作peer网卡时候，比如将peer网卡放入netns时候，有几种方法：

1. 用nic.Nic来初始化这个peer：

	```go
	peername := "poi-xxx"
	peer := &nic.Nic{
	    Name: peername,
	    Netns: ""
	}
	peer.SetNetns("ns1")
	```

2. 和上面类似，但是在nic.go里再搞一个New()方法用于处理这种既不属于veth、也不属于bridge、也不属于vtep的独立普通网卡

3. 自己新搞一个struct，比如叫normalNic来处理这种独立网卡

4. 在veth.go里新增`func (this *veth) SetPeerNetns(ns string)`

5. 在veth.go里新增`func SetPeerNetns(peer string, ns string)`

6. 在veth.go里新增`func SetPeerNetns(v *veth) { ip link set v.peer netns }`

个人推荐用4最好，既不会为此把nic基类搞臃肿，同时又作为veth的方法存在（不管怎么说peer也是自家人，没有当作外乡人）

接下来想做一个函数，传递参数给这个函数，然后这个函数帮我调用bridge、 veth、vtep，然后返回指针struct（返回参数是实现了vnic interface），但是发现不对：

interface的使用要满足2个条件才有意义：

1. 实现了interface的几个struct是平级的，并且输入输出参数完全一致。（这点是interface的本质，能实现interface的肯定是满足这个条件）

2. 在业务逻辑上，调用实现interface的struct是不定的，而不是顺序逻辑，比如这样是不对的，因为netns_struct、bridge_struct、veth_struct是有顺序的：

```go
func main() {
    var i vnic
    i = &netns_struct{...}
    i.Add()
    i = &bridge_struct{...}
    i.Add()
    i = &veth_struct{...}
    i.Add()
}
```

这样逻辑是正确的：

```go
var i vnic
switch opt {
case "netns":
    i = &netns.struct{}
case "bridge":
    i = &bridge.struct{}
case "veth":
    i = &veth.struct{}
}
i.Add()
i.Del()
```

就是说调用者这方对于实现interface的struct是根据某个参数（通过API传递过来，或者配置文件传递过来，或者etcd传递过来）来选择某个struct，这种逻辑才适用interface。而在vrouter程序逻辑里，netns、bridge、veth是被调用者依次执行，因此不适用interface。

```go
type vnic interface {
    Add()
    Del()
    Up()
    Down()
    SetMaster(string)
    SetNetns(string)
    AddIPAddr(string)
    IsExists()
}
```

所以上面这种定义接口的思路就是错误的，这个例子中暂时用不到接口
