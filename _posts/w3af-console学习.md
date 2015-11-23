title: w3af_console学习
date: 1912-12
categories: [kali]
tags: [kali,w3af]
---
Kali2.0 路径 /usr/share/w3af/
[官网 http://w3af.org](http://w3af.org/)
> git clone https://github.com/andresriancho/w3af.git

查找XSS与SQLi
cd w3af
./w3af_gui
#### 第一步
设置“target URL”
#### 第二部
“empty_profile”
配置w3af 选择“crawl” plugins
启用 “web_spider” plugin
#### 第三部
设置完成后点击“crawl”

w3af不会标记XXS或SQLi，除非设置了 “audit” lugins

“output” plugins 一般情况下是导出text文本，
如果需要也可以导出 XML 格式

    w3af>>> plugins
    //进入插件模块
    w3af/plugins>>> list discovery 
    //列出所有用于发现的插件
    w3af/plugins>>> discovery findBackdoor phpinfo webSpider 
    //启用findBackdoor phpinfo webSpider这三个插件
    w3af/plugins>>> list audit 
    //列出所有用于漏洞的插件
    w3af/plugins>>> audit blindSqli fileUpload osCommanding sqli xss 
    //启用blindSqli fileUpload osCommanding sqli xss这五个插件
    w3af/plugins>>> back
    //返回主模块
    w3af>>> target
    //进入配置目标的模块
    w3af/config:target>>> set target http://192.168.244.132/
    //把目标设置为http://192.168.244.132/
    w3af/config:target>>> back
    //返回主模块
    w3af>>>start

    w3af>>> exploit 
    //进入漏洞利用模块
    w3af/exploit>>> list exploit
    //列出所有用于漏洞利用的插件
    w3af/exploit>>> exploit sqlmap 
    //使用sqlmap进行SQL注入漏洞的测试
