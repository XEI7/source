title: Wireshark过滤器
date: 1912-12
categories: [网络]
tags: [wireshark,笔记]
---
### Wireshark 过滤器常用的命令:
介绍：由于有可能在短时间内抓包特别多，数据不利于分析，这时我们就需要用过滤器来显示我们所需要显示的内容，我们本章主要讲的也是过滤器的一些简单的使用方法，会用过滤器是一门手艺，大家如果想深入的话还需要自己购买书籍多深入研究才行.


<!--more-->

#### 过滤器的运算符
 Comparison operators （比较运算符）:
可以使用6种比较运算符：
英文写法： 	C语言写法： 	含义：
eq 	== 	等于
ne	!=	不等于
gt	>	大于
lt	<	小于
ge	>=	大于等于
le	<=	小于等于
Logical expressions（逻辑运算符）:
英文写法： 	C语言写法： 	含义：
and	&&	逻辑与
or	||	逻辑或
xor	^^	逻辑异或
not	!	逻辑非


#### IP的一些简单命令:
##### IP过滤来源过滤：
当我们需要寻找本是什么地址发出的请求（即“来源”）时就可以使用如下过滤器命令
IP.src == 192.168.1.1 格式: 协议类型 | 子类 | 运算符 IP 
 

##### IP过滤目的的过滤：
当我们需要寻找本目标发出的请求的目的（即“目的”）时就可以使用如下过滤器命令
IP.dst == 192.168.1.1 格式: 协议类型 | 子类 | 运算符 IP 
 

##### IP 寻找非指定IP的包：
当我们想寻找IP非来源或者目的IP包时可使用如下命令进行过滤：
!(ip.addr == 10.31.10.5)
 
##### 学会巧妙组合:
如果想只显示指定的来源ip 和 指定的目的 ip 可以组合使用
ip.src eq 10.31.10.4 and ip.dst == 239.255.255.250
 
等等还有很多其他的组合大家后期可以自行组合测试进行实验。

#### TCP端口的过滤
##### 普通的tcp查询：
Tcp的查询格式为 TCP | 子类 | 运算符 | 端口
已查询80端口为列 tcp.port == 80
 
其端口为80
  
还能查来源或者目的端口为80的实例分别为
Tcp.dstport == 80
Tcp.srcport == 80
子类还有flags 控制位，ACK序列位，seq，windows_size位等…大家可以自己去尝试

##### 进行TCP的组合查询
Tcp和其他一样也可以进行组合查询你可以用他查询ip为XXX，端口为XX等事例进行精确查找.方式如下
Ip.dst == 61.135.189.35 And Tcp.dstport == 80
或者排除它，还有更多组合等你发现
!(ip.dst == 61.135.189.35 and tcp.dstport == 80)
4.4 进行mac地址过滤
格式：eth | 目标 | 运算符 | MAC |
目标:
eth.dst == 18-03-73-d0-99-34
来源:
eth.src == 18-03-73-d0-99-34
来源和目标同等于mac
eth.addr == 18-03-73-d0-99-34
eth.addr[0:3]==00:1e:4f 搜索过滤MAC地址前3个字节是0x001e4f的数据包。

#### HTTP过滤
##### 查询包含特性的http连接 :
格式 http | 参数
http.accept 包含文件传输http连接
http.cookie 查看带cookie的数据帧
进阶格式: http | 命令 | 运算符 | 参数
http.cookie == “412412” 查看符合此COOKIE特性
##### 其他特性方式
格式：http | 类型 | 特性
http contains “GET”
http contains “POST”
查询HTTP协议版本也如此
http contains “HTTP/1.1”
http contains “HTTP/1.0”
##### 其他查询:
格式：http | 类型 | 类型 | 运算符 | 参数  
http.request.method == “POST”
http.request.method == “GET”
http | 命令 | 命令 | 运算符 | 参数 |
http.request.uri == “/so/t.gif” http.referer好像类似，都好像绝对值？
http.request.uri contains "data"  这个比较好用，搜索类似带data的数据帧
//GET包 特性！
http.request.method == "GET" && http contains "Host:"  （主机名同城为主域）
http.request.method == "GET" && http contains "User-Agent"  用户代理（系统环境）
http.request.method == "GET" && http contains "cookie:"  
提交方式为get   并且 包含cookie 的内容
http.request.method == "GET" && http contains "163.com" 获取host为163的数据帧等
//POST包
http.request.method  =="POST" && http contains "host:" 同理，只是改了POST
//响应包
http contains "HTTP/1.1 200 OK" && http contains "Content-Type:”
http contains "HTTP/1.0 200 OK" && http contains "Content-Type:"
HTTP/1.1 与1.0分别为版本    " Content-Type:"
包含Content-Type: 肯定有附带数据包
