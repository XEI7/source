title: 黑客攻防技术宝典web实战篇笔记
date: 2016-01-05 11:40:15
categories: [web]
tags: [web]
---

黑客攻防技术宝典web实战篇
<!--more-->


P79
1.配置浏览器使用Burp或WebScarab作为本地代理
2.以常规方式浏览整个应用程序，访问发现的每一个URL，提交每一个表单并执行全部多阶段功能。尝试启用或禁用JavaScript、cookie的情况下进行浏览。
3.检查有代理服务器或爬虫工具生成的站点地图，确定手动浏览是没有发现的所有应用程序内容或功能。

p83  【蛮力测试】
1.手动提出一些访问有效与无效的资源的请求，并确定服务器如何处理无效资源
2.使用用户指定的抓取生成的站点地图作为自动查找隐藏内容基础。
3.Burp Intruder结合字典，迅速生成大量请求。
4.收集从服务器收到的响应，并手动检查这些响应以确定有效的资源
5.反复执行这个过程，知道发现新内容。

p84
1.检查用户指定的浏览与基本蛮力测试获得的结构。编译枚举出的所有子目录名称、文件词干和文件扩展名列表。
2.检查这些列表，确定应用程序使用的所有命名方案。


4.2.4
p102
1.客户端确认--服务器没有采用确认检查
2.数据库交互--SQL注入
3.文件上传与下载--路径遍历漏洞、保存型跨站点脚本
4.显示用户提交的数据--跨站点脚本
5.动态重定向--重定向与消息头注入攻击
6.社交网络功能--用户名枚举、保存型跨站点脚本
7.登录--用户名枚举、脆弱密码、能使用蛮力
8.多阶段登录--登录缺陷
9.会话状态--可推测出的令牌、令牌处理不安全
10.访问控制--水平权限和垂直权限提升
11.用户伪装功能--权限提升
12.使用明文通信--会话劫持、收集证书和其他敏感数据
13.站外链接--Referer消息头中查询字符串参数泄漏
14.外部系统接口--处理会话与/或访问控制的快捷方式
15.错误消息--信息泄漏
16.电子邮件交互--电子邮件与命名注入
17.本地代码组件或交互--缓冲区溢出
18.使用第三方应用程序组件--已知漏洞
19.已确定的web服务器软件--常见配置薄弱环节、已知软件程序缺陷。

5.1.3
p109
URL参数


p111
模糊数据实施
1.如果知道模糊字符串的明文值，可以尝试破译模糊处理所使用的模糊算法
2.应用程序的2其他地方可能包换一些功能，攻击者可以利用他们返回由自己控制的一段明文生成的模糊字符串。在这种情况下，攻击者可以向目标功能直接提交任意一个有效载荷，获得所需要的字符串。
3.即使模糊字符串完全无法理解，也可以在其他

#### 收集用户数据：HTML表单
p115
1.寻找包含maxlength属性的表单元素。提交大于这个长度但其他格式合法的数据（）
2.如果应用程序接受的这个超长的数据，则可以据此推断出服务器并没有采用客户端确认机制
3.根据应用程序随后对参数进行的处理，可以通过确认机制中存在的缺陷利用其他漏洞，如SQL XSS 缓冲区溢出。
#### 基于脚本的确认
1.确定任何在提交表单前使用客户端javascript进行输入确认的情况
2.通过修改所提交的请求，再其中插入无效数据，或者修改确认代码使其失效，向服务器提交确认机制通常会阻止的数据
3.与长度限制一样，确定服务器是否采用了和客户端相同的控件；如果并非如此，确定是否可以利用这种情况实现任何恶意意图
4.注意，如果在提交表单前有几个输入字段需要由客户端确认机制校验，需要分别用无效数据此时每一个字段，同事再多有其他字段中使用有效数据。如果同时再几个字段中提交无效数据，可能服务器再识别出第一个无效字段时就已经停止执行表单，从而使测试无法达到应用程序的所有可能代码路径。
#### 禁用元素
1.在应用程序的每一个表单中需找禁用的元素。尝试将发现的每一个元素与表单的其他参数一起提交给服务器，确定其是否有效。
2.通常，如果提交元素被标记为禁用，其按钮即以灰色显示，表示相关操作无效。这时应该尝试提交这些元素的名称，确定应用程序是否再执行所有请求的操作前执行服务器端检查。
3.注意，再提交表单时，浏览器并不包含禁用的表单元素；因此，仅仅通过浏览应用程序的功能以及监控由浏览器发布的请求并不能确定其中是否含有禁用元素。要确定禁用的元素，必须监控服务器的响应或在浏览器中查看页面来源。
4.还可以使用Burp Proxy 中的HTML修改功能自动重新启用应用程序中的任何禁用的字段
### 5.3 收集用户数据：浏览器扩展  P118











http://www.ms509.com/