title: sqlmap学习2
categories: [sqlmap]
date: 2015-01-03 14:26:39
tags: [sqlmap]
---

文章摘要
<!--more-->
#### SQLMAP的准备工作

SQLMAP在进行扫描之前会有防火墙的检测，当检测到有防火墙之后，进行SQL检测时的判断依据也会有所调整，如bool类型的盲注的时候，而其中heuristicCheckSqlInjection这一函数既会影响到接下来所要使用什么样的Payload进行测试，heuristicCheckSqlInjection翻译过来的意思是启发式SQL注入测试，那麽到底什么是启发式，
提示的依据就是根据这一个启发式SQL注入测试也就是本文要介绍的heuristicCheckSqlInjection来决定的。SQLMAP中有许多的Payload，足以有几百条，那麽如果全部Payload都测试一遍的话，无疑是一件很浪费时间的事情，除非Payload中的risk，clause，where字段中加以过滤，还会以这一个根据数据库的指纹来选择所要测试的Payload。

heuristicCheckSqlInjection的主要作用可以分为以下几点:

1.数据库版本的识别。
2.绝对路径获取。
3.XSS的测试

#### 数据库版本的识别

# Alphabet used for heuristic checks
HEURISTIC_CHECK_ALPHABET = ('"', '\'', ')', '(', ',', '.')
...
randStr = ""
while '\'' not in randStr:
    randStr = randomStr(length=10, alphabet=HEURISTIC_CHECK_ALPHABET)
kb.heuristicMode = True
payload = "%s%s%s" % (prefix, randStr, suffix)
payload = agent.payload(place, parameter, newValue=payload)
page, _ = Request.queryPage(payload, place, content=True, raise404=False)
...
首先会从HEURISTIC_CHECK_ALPHABET中随机抽取10个字符出现构造Payload，当然里面的都不是些普通的字符，而且些特殊字符，当我们进行SQL注入测试的时候会很习惯的在参数后面加个分号啊什么的，又或者是其他一些特殊的字符，出现运气好的话有可能会暴出数据的相关错误信息，而那个时候我们就可以根据所暴出的相关错误信息去猜测当前目标的数据库是什么。

实际找个网站测试，打下码，保护下。

http://***.***.***/datalist/default.aspx/article?category_id=1051
那麽经过while '\'' not in randStr:后生成了随机的字符，然后就是发包检测返回的数据了。

 

其实熟悉SQL注入的人也知道这是一个Oracle的一个错误信息，那麽接下来看看SQLMAP到底是怎样去判断的。

位于./lib/request/connect.py中的getPage函数,大约在598行。

def getPage(**kwargs):
    ...
    processResponse(page, responseHeaders)
    ...
其中processResponse会调用到./lib/parse/html.py中的htmlParser函数，这一个函数就是根据不同的数据库指纹去识别当前的数据库究竟是什么。

def htmlParser(page):
    """
    This function calls a class that parses the input HTML page to
    fingerprint the back-end database management system
    """
    xmlfile = paths.ERRORS_XML
    handler = HTMLHandler(page)
    parseXmlFile(xmlfile, handler)
    if handler.dbms and handler.dbms not in kb.htmlFp:
        kb.lastParserStatus = handler.dbms
        kb.htmlFp.append(handler.dbms)
    else:
        kb.lastParserStatus = None
    # generic SQL warning/error messages
    if re.search(r"SQL (warning|error|syntax)", page, re.I):
        handler._markAsErrorPage()
    return handler.dbms
最终实现的的其实是HTMLHandler这个类，而paths.ERRORS_XML这一变量的就是SQLMAP用来识别的指纹配置文件路径，位置在于./xml/errors.xml中。
 
<!-- Oracle -->
<dbms value="Oracle">
    <error regexp="\bORA-[0-9][0-9][0-9][0-9]"/>
    <error regexp="Oracle error"/>
    <error regexp="Oracle.*Driver"/>
    <error regexp="Warning.*\Woci_.*"/>
    <error regexp="Warning.*\Wora_.*"/>
</dbms>
这一配置文件的比较简单，其实也就是一些对应数据库的正则。SQLMAP在解析errors.xml的时候，然后根据regexp中的正则去匹配当前的页面信息然后去确定当前的数据库。

class HTMLHandler(ContentHandler):
    """
    This class defines methods to parse the input HTML page to
    fingerprint the back-end database management system
    """
    def __init__(self, page):
        ContentHandler.__init__(self)
        self._dbms = None
        self._page = page
        self.dbms = None
    def _markAsErrorPage(self):
        threadData = getCurrentThreadData()
        threadData.lastErrorPage = (threadData.lastRequestUID, self._page)
    def startElement(self, name, attrs):
        if name == "dbms":
            self._dbms = attrs.get("value")
        elif name == "error":
            if re.search(attrs.get("regexp"), self._page, re.I):
                self.dbms = self._dbms
                self._markAsErrorPage()
可以发现当前返回的页面信息命中了<error regexp="\bORA-[0-9][0-9][0-9][0-9]"/>这一条正规

Sqlmap3

 

到此SQLMap就可以确定数据的版本了，从而选择对应的测试Payload，减少SQLMAP的扫描时间。

#### 绝对路径获取与XSS检测

相比指纹识别，获取绝对路径的功能模块相对简单，利用正则匹配寻找出绝对路径。

def parseFilePaths(page):
    """
    Detects (possible) absolute system paths inside the provided page content
    """
    if page:
        for regex in (r" in (?P.*?) on line", r"(?:>|\s)(?P[A-Za-z]:[\\/][\w.\\/]*)", r"(?:>|\s)(?P/\w[/\w.]+)"):
            for match in re.finditer(regex, page):
                absFilePath = match.group("result").strip()
                page = page.replace(absFilePath, "")
                if isWindowsDriveLetterPath(absFilePath):
                    absFilePath = posixToNtSlashes(absFilePath)
                if absFilePath not in kb.absFilePaths:
                    kb.absFilePaths.add(absFilePath)
而XSS的检测代码位于889行中:

# String used for dummy XSS check of a tested parameter value
DUMMY_XSS_CHECK_APPENDIX = "<'\">"
    ...
    value = "%s%s%s" % (randomStr(), DUMMY_XSS_CHECK_APPENDIX, randomStr())
    payload = "%s%s%s" % (prefix, "'%s" % value, suffix)
    payload = agent.payload(place, parameter, newValue=payload)
    page, _ = Request.queryPage(payload, place, content=True, raise404=False)
    paramType = conf.method if conf.method not in (None, HTTPMETHOD.GET, HTTPMETHOD.POST) else place
    if value in (page or ""):
        infoMsg = "heuristic (XSS) test shows that %s parameter " % paramType
        infoMsg += "'%s' might be vulnerable to XSS attacks" % parameter
        logger.info(infoMsg)
    ...
最后根据输入的字符是否留着页面上，如果存在就提示有可能拥有XSS漏洞。

#### 总结

至此heuristicCheckSqlInjection的功能也介绍的差不多了，通过具体了解SQLMAP的一些扫描规则又或者是思路，可以让我们根据具体的情况去配置SQLMAP又或者编写自己的SQL Fuzz系统，其中可以通过编辑errors.xml这一指数据纹配置来增强SQLMAP的嗅探能力，从而打造更强大的神器了。




一般的操作是：先把proxy的requests请求保存成一个log文件，然后执行命令：
python sqlmap.py -l requests.log --smart --batch
一般有SQL注入漏洞的都啪啪啪的挖出来的，但是这里有个问题，因为 log是 requests的请求log所以就包含了所有的 requests请求。像静态资源文件啊（js.html.css）什么的都包含在里面，这就会导致整个检测周期会变得比较长。
    本着提高检测效率的精神，咱对sqlmap做一下简单的更改，这也完全得意于sqlmap作者对其开源。
修改比较简单，对请求的url进行判断，使用正则表达式对url内容进行判断，如果匹配的话就不往下执行。

-l 参数修改：
在\sqlmap\lib\controller\controller.py文件找到 def start()，在conf.cookie = targetCookie加入以下代码
```
    for targetUrl, targetMethod, targetData, targetCookie, targetHeaders in kb.targets:
        try:
            conf.url = targetUrl
            conf.method = targetMethod.upper() if targetMethod else targetMethod
            conf.data = targetData
            conf.cookie = targetCookie

            #''' Static resource not injection '''
            Static = re.findall(r"\w+\.(?:js|html|css|jpg|png)\?\w+",conf.url)
            if Static:
                infoMsg = "Static resource: %s not injection" % Static
                logger.info(infoMsg)
                break
            #''' Static resource not injection '''

            conf.httpHeaders = list(initialHeaders)
            conf.httpHeaders.extend(targetHeaders or [])

            initTargetEnv()
```

-u参数修改：
在\sqlmap\lib\controller\checks.py 文件找到 def checkConnection(suppressOutput=False) ，在if not suppressOutput and not conf.dummy加入以下代码

```
    if not suppressOutput and not conf.dummy and not conf.offline:
        infoMsg = "testing connection to the target URL"
        logger.info(infoMsg)

    #''' Static resource not injection '''
    Static = re.findall(r"\w+\.(?:js$|html|css|jpg|png)",conf.url)
    if Static:
        infoMsg = "Static resource: %s not injection" % Static
        logger.info(infoMsg)
        return False
    #''' Static resource not injection '''
    
    try:
        kb.originalPageTime = time.time()
```
 
修改完后，运行如下：
 
python sqlmap.py -l 1.log -v 3



先听我讲sqlmap源代码有个小bug，导致不能全量检测会漏掉一些URL和参数。
具体源代码如下：lib/core/option.py中_parseBurpLog()模块代码第366行，如下图：
当kb.targets不为空或url不在addedTargetUrls里时就添加检测队列，这会导致什么问题呢？
例如：
#1：
post /1.php
a=ok&b=ok
#2：
post /1.php
a=ok&c=inject

两个url相同#1的参数不存在注入，#2的c参数存在注入。但sqlmap在批量提取url时由于两个url相同，所以只取了#1，而#2因为url跟#1的url一样，所以没添加到检测队列里。就这样我们的sqlmap就漏测#2存在注入的c参数了（真实被坑过）。

基于以上原因，我决定对sqlmap加入检测队列算法进行修造，加入检测队列在url的基础上加入data的key做为判断条件（url+key），这样相同url而key不同请求history记录sqlmap也不漏测。
首先，把data里的Key提取出来，并对key进行一次按字符顺序排序，然后跟url组合是否加入检测队列的判断条件。
具体实现代码如下：
```
if not(conf.scope and not re.search(conf.scope, url, re.I)):
            #处理data参数，把key提取出来，key值做为判断重复的条件之一
            a = data
            b = a.split('&')
            #print b
            c = []
            for i in b:
                if i.find('=') > 0:
                    arr = i.split('=')
                    c.append(arr[0])
            #print c
            #提取的key使用sorted按字符顺序排序一把，再组合成字符串。(如果数据庞大的话，建议把keystr转成hash值)
            keystr = "%s%s"%(url,''.join(sorted(c)))
            #if not kb.targets or url not in addedTargetUrls:
            if not kb.targets or keystr not in addedTargetUrls:
                kb.targets.add((url, method, data, cookie, tuple(headers)))
                #addedTargetUrls.add(url)
                addedTargetUrls.add(keystr)

```