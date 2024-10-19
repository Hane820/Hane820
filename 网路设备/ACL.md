## 概念
---
**访问控制列表--ACL（Access Control List）是应用在路由器接口的指令列表，这些指令列表用来告诉路由器，哪些数据包可以接收，哪些数据包需要拒绝。它的主要任务是保证网络资源不被非法使用和访问，是保证网络安全最重要的核心策略之一。**

>==*请注意，路由器本身发送的数据不受ACL控制*==

## 原理
---
**ACL使用包过滤技术，在路由器上读取OSI七层 模型的第三层（网络层）及第四层（传输层）
包头中的信息。如：源地址，目的地址，源端口，目的端口，根据预先定义好的规则，对包进行过滤，从而达到访问控制的目的。

## 作用
---
1. ACL可以限制网络流量，提高网络性能
2. ACL是提供网络安全访问的基本手段
3. ACL可以在路由器端口处决定哪种类型的通信流量被转发或阻塞

## ACL应用原则以及执行过程
### 原则
---
**3P原则:每种协议(per protocol),每个方向(per direction),每个端口(per interface)配置一个ACL**

**一个==标准ACL==匹配IP包中的源地址或源地址中的一部分，可对匹配的包采取拒绝或允许两个操作，且标准访问控制列表要尽量靠近目的端。**


### 执行过程
---
一个端口执行哪条ACL，这需要按照列表中的条件语句执行顺序来判断。如果一个数据包的报头跟表中某个判断语句相匹配，name后面的雨具就将忽略，不再检查
> **省流：switch 的每个case里都有一个break;

数据包只有在第一个判断条件不匹配时，它才被交给ACL中下一个条件判断语句进行比较。如果匹配（假设允许发送），则不管是第一条还是最后一条语句，数据都会立即发送到目的接口。如果所有的ACL判断语句都检测完毕，仍然没有匹配的语句，则该数据包将视为拒绝而被丢弃。（也就是默认最后一条语句为拒绝发送）
>**省流： default false;

## 分类
目前主要有三种ACL：
1. 标准ACL
	**标准ACL使用1-99以及1300-1999之间的数字作为表号，是基于==源IP地址==过滤数据包**
2. 扩展ACL
	**扩展ACL使用100-199以及2000-2699之间的数字作为表号，可以基于==源IP地址，目的IP地址，指定协议，端口和标志来过滤数据包==
3. 命名ACL
	命名ACL是以列表名称代替编号来定义ACL，同样包括标准和扩展两种列表。在使用命名访问控制列表时，要求路由器IOS是11.2以上的版本，并且不能以同一个名字命名多个ACL，不同类型的ACL也不能相同名字
	>**省流：primary_key unique
4. 时间ACL
	从IOS12.0开始，思科路由器新增加了一种基于时间的访问系统列表。可以根据一天中不同时间，或是日期，来控制网络数据包转发
## 定义规范
1. ACL是通过表号或者表名区别的
2. 一个ACL的配置是[3P](#原则)的
3. ACL的雨具顺序决定了数据包的控制顺序
4. 最有限制性的雨具应该放在ACL语句 的首行
5. 在将ACL应用到接口之前，一定要先建立ACL
6. 在ACL的最后，有一条隐含“全部拒绝”的命令，所以在ACL是至少有一条“允许”的语句。
7. ACL职能过滤穿过路由器的数据流量，不能过滤本路由器上发出的数据包
8. 在路由器进行选择以前，应用在接口进入方向的ACL起作用
9. 在路由器选择决定以后，用在交界口离开的方向的ACL起作用

## ==标准ACL配置命令==
### 创建ACL

Router(config)#access-list access-list==[number]== ==permit | deny== ==source-ip-address== ==wildcard-mask==
							列表编号	允许或拒绝	源IP地址	子网掩码反码
eg.`Rouuter(config)#access-list access-list 1 permit 192.168.1.0 0.0.0.255

### 将ACL应用于接口

Router(config-if)#ip access-group access-list-number in | out
eg.`Router(config-if)#ip access-group 1 in`

### 删除已建立的标准ACL

Router(config-if)#ip access-group access-list-number in | out
eg. 'Router(config-if)#ip access-group 1 in'
>对于标准ACL，不能删除单条ACL语句，只能够删除整个ACL


## 扩展ACL配置命令
### 创建ACL


> Router(config)#access-list *access-list-number*<sup>1</sup>*permit| deny*<sup>2</sup> *protocol* <sup>3</sup>source *source-wildcard* *destion<sup>4</sup> destion-wildcard*  <sup>5</sup>*operator operan*<sup>6</sup>

[1]:访问控制列表号
[2]:允许或拒绝
[3]:协议类型，如IP/TCP/UDP/ICMP
[4]:源反码
[5]:目的反码
[6]:  it（小于），gt（大于），eq（等于）或neq（不等于）一个端口号。

### 将ACL应用到特定的端口
`Router(config-if)#ip access-group access-list-number in | out`

## 举例

**eg.允许网络192.168.1.0/24访问网络192.168.2.0/24的==IP流量通过==，而拒绝其他任何流量，ACL配置命令如下：**
 `Router(config)#access-list 101 permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255`
`Router(config)#access-list 101 deny ip any any`

**eg.拒绝网络192.168.1.0/24==访问FTP服务器192.168.2.2/24的IP流量通过==，而允许其他任何流量，ACL配置命令如下：**

`Router(config)#access-list 101 deny tcp 192.168.1.0 0.0.0.255 host 192.168.2.2 eq 21
`Router(config)#access-list 101 permit ip any any

**eg.禁止网络192.168.1.0/24中的==主机ping通服务器==192.168.2.2/24，而允许其他任何流量，ACL配置命令如下：**

`Router(config)#access-list 101 deny icmp 192.168.1.0 0.0.0.255 host 192.168.2.2 echo
`Router(config)#access-list 101 permit ip any any

**eg.禁止A网段192.168.1.1-192.168.1.30访问B网段的主机192.168.2.200。**
`Router(config)#access-list 100 deny ip 192.168.1.0 0.0.0.31 host 192.168.2.200`
`Router(config)#access-list 100 permit ip any any`
`Router(config)#int fa0/0`
`Router(config-if)#ip access-group 100 in`



## 命名ACL