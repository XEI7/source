title: sqlmap学习
categories: [sqlmap]
date: 2015-01-03
tags: [sqlmap]
---

SQL扫描规则
<!--more-->
要了解SQLMap的扫描规则，也就是Payload ，SQLMap的扫描规则文件位于\xml文件夹中，其中boundaries.xml与Payloads文件夹则为SQLMap的扫描规则所在，\xml\payloads中的6个文件，里面的6个文件分别是存放着不同注入手法的PAYLOAD。
那么就必须了解两个格式，一是boundary文件，一是payloads。

例子：
```
<boundary>    
<level>1</level>    
<clause>1</clause>    
<where>1,2</where>    
<ptype>1</ptype>    
<prefix>'</prefix>    
<suffix> AND '[RANDSTR]'='[RANDSTR]</suffix>
</boundary>
```
1. clause与where属性

这两个元素的作用是限制boundary所使用的范围，可以理解成当且仅当某个boundary元素的where节点的值包含test元素的子节点，clause节点的值包含test元素的子节点的时候，该boundary才能和当前的test匹配，从而进一步生成payload。

2. prefix与suffix属性

要理解这两个属性的作用，那麽就先利用一段代码去讲解。

function getattachtablebypid($pid) {
   $tableid = DB::result_first("SELECT tableid FROM ".DB::table('forum_attachment')." WHERE pid='$pid' LIMIT 1");
   return 'forum_attachment_'.($tableid >= 0 && $tableid < 10 ? intval($tableid) : 'unused');
}
通过代码我们可以知道pid参与了SQL语句的拼接，那麽如果我们输入的pid为' AND 'test' = 'test呢，那麽最终拼接起来的SQL语句应该为：

SELECT tableid FROM ".DB::table('forum_attachment')." WHERE pid='' AND 'test' = 'test' LIMIT 1
所以如果我们输入的是' AND 'test' = 'test，那麽最终拼接起来的SQL语句同样是合法的。那麽我们就可以把所测试的Payload放到prefix与suffix中间，使之最终的SQL合法，从而进行注入测试，所以通过了解，prefix与suffix的作用就是为了截断SQL的语句，从而让最终的Payload合法。
至此boundary文件的作用已经讲解完了，接下来就是payload的讲解了。
```
<test>    
<title>MySQL &gt;= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause</title>    
<stype>2</stype>    
<level>1</level>    
<risk>1</risk>    
<clause>1,2,3</clause>    
<where>1</where>    
<vector>AND (SELECT [RANDNUM] FROM(SELECT COUNT(*),CONCAT('[DELIMITER_START]',([QUERY]),'[DELIMITER_STOP]',FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a)</vector>    
<request><!-- These work as good as ELT(), but are longer<payload>AND (SELECT [RANDNUM] FROM(SELECT COUNT(*),CONCAT('[DELIMITER_START]',(SELECT (CASE WHEN ([RANDNUM]=[RANDNUM]) THEN 1 ELSE 0 END)),'[DELIMITER_STOP]',FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a)</payload><payload>AND (SELECT [RANDNUM] FROM(SELECT COUNT(*),CONCAT('[DELIMITER_START]',(SELECT (MAKE_SET([RANDNUM]=[RANDNUM],1))),'[DELIMITER_STOP]',FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a)</payload>-->
<payload>AND (SELECT [RANDNUM] FROM(SELECT COUNT(*),CONCAT('[DELIMITER_START]',(SELECT (ELT([RANDNUM]=[RANDNUM],1))),'[DELIMITER_STOP]',FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a)</payload>
</request>    
<response>        
<grep>[DELIMITER_START](?P&lt;result&gt;.*?)[DELIMITER_STOP]</grep>    
</response>    
<details>        
<dbms>MySQL</dbms>        
<dbms_version>&gt;= 5.0</dbms_version>    
</details>
</test>
```
1. title属性

title属性为当前测试Payload的标题，通过标题就可以了解当前的注入手法与测试的数据库类型。

2. stype属性

这一个属性标记着当前的注入手法类型，1为布尔类型盲注，2为报错注入。

3. level属性

这个属性是每个test都有的，他是作用是是限定在SQL测试中处于哪个深度，简单的来说就是当你在使用SQLMAP进行SQL注入测试的时候，需要指定扫描的level，默认是1，最大为5，当level约高是，所执行的test越多，如果你是指定了level5进行注入测试，那麽估计执行的测试手法会将超过1000个。

4. clause与where属性

test中的clause与where属性与boundary中的clause与where属性功能是相同的。

5. payload属性

这一属性既是将要进行测试的SQL语句，也是SQLMap扫描逻辑的关键，其中的[RANDNUM]，[DELIMITER_START]，[DELIMITER_STOP]分别代表着随机数值与字符。当SQLMap扫描时会把对应的随机数替换掉，然后再与boundary的前缀与后缀拼接起来，最终成为测试的Payload。

6. details属性

其子节点会一般有两个，其dbms子节所代表的是当前Payload所适用的数据库类型，当前例子中的值为MySQL，则表示其Payload适用的数据库为MySQL，其dbms_version子节所代表的适用的数据库版本。

7. response属性

这一属性下的子节点标记着当前测试的Payload测试手法。

        grep        ：报错注入
        comparison  ：布尔类型忙注入
        time        ：延时注入
        char        ：联合查询注入
SQLMAP当中的checkSqlInjection函数即是用这一属性作为判断依据来进入不同的处理分支。而且其中response属性中的值则为其SQL注入判断依据，就如当前的例子中，grep中的值为[DELIMITER_START](?P&lt;result&gt;.*?)[DELIMITER_STOP]，SQLMap会将[DELIMITER_START]与[DELIMITER_STOP]替换成Payload中所对应替换的值，然后利用所得到的对返回的页面信息进行正则匹配，如果存在在判断为当前存在SQL注入漏洞。

其中要注意的是，Payload中的字符串会根据当前Payload所适用的数据库类型对字符串进行处理，其处理的代码位于\plugins\dbms下对应数据库文件夹中的syntax.py脚本中。

14419658766867

 

所以最终的payload是根据test的payload子节点和boundary的prefix（前缀）、suffix（后缀）子节点的值组合而成的，即：最终的payload =  url参数 + boundary.prefix+test.payload+boundary.suffix

三、实例

接下来以报错注入来实际讲解下Payload与boundary的使用。

上例子中的boundary元素中的where节点的值为1，2，含有test元素的where节点的值（1），并且，boundary元素中的clause节点的值为1，含有test元素的where节点的值（1），因此，该boundary和test元素以匹配。test元素的payload的值为：

AND (SELECT [RANDNUM] FROM(SELECT COUNT(*),CONCAT('[DELIMITER_START]',(SELECT (CASE WHEN ([RANDNUM]=[RANDNUM]) THEN 1 ELSE 0 END)),'[DELIMITER_STOP]',FLOOR(RAND(0)*2))x FROM information_schema.tables GROUP BY x)a)
之前已经介绍了最终的Payload是如何的一个格式，所以最后将其中的[RANDNUM]、[DELIMITER_START]、[DELIMITER_STOP]替换掉与转义之后。

则生成的payload类似如下：

[RANDNUM]           = 2214
[DELIMITER_START]   = ~!(转义后则为0x7e21)
[DELIMITER_STOP]    = !~(转义后则为0x217e)

Payload: ' AND (SELECT 2214 FROM(SELECT COUNT(*),CONCAT(0x7e21,(SELECT (CASE WHEN (2214=2214) THEN 1 ELSE 0 END)),0x217e,FLOOR(RAND(0)*2))x FROM information_schema.tables GROUP BY x)a) AND 'pujM'='pujM
如果http://127.0.0.1/search-result.php?keyword=&ad_id=3存在注入的话，那么执行的时候就会报如下错误：

Duplicate entry '~!1!~1' for key 'group_key'
根据之前的讲解，那么最终于测试的URL如下：

http://127.0.0.1/search-result.php?keyword=&ad_id=' AND (SELECT 2214 FROM(SELECT COUNT(*),CONCAT(0x7e21,(SELECT (ELT(2214=2214,1))),0x217e,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a) AND 'YmRM'='YmRM
 
最后根据grep中的正规来匹配当前页面。

<grep>[DELIMITER_START](?P&lt;result&gt;.*?)[DELIMITER_STOP]</grep>
而使用正则：~!(?P<result>.*?)!~来匹配Duplicate entry '~!1!~1' for key 'group_key' 的结果为1,根据匹配的结果可以得出当前的页面确实存在着SQL注入。

总结

通过SQLMap的扫描逻辑，我们可以了解到SQL注入的常规手法与实现，熟悉SQLMap的配置文件之后，自己就可以根据实际的情况对Payload与boundary进行修改，通过增加Payload与boundary来增强SQLMap的扫描规则，也可以利用其扫描规则来打造一款自己的SQL扫描工具。










## 首先详细看一下payload中字段：
#### title字段：
payload test起的名字； 譬如我们给自己的payload起的名字为：

```	
MySQL boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause (desc-mayikissu)
```
#### stype字段：
sql注入的类型sql注入分了如下类型：

    类型1：盲注 我们order by类型属于盲注，因此我们添加stype的值为1. 
    类型2：错误类型注入 
    类型3：内联注入 
    类型4：多语句查询注入 
    类型5：时间注入 
    类型6：联合查询注入

#### level字段：
sqlmap对于每一个payload都有一个level级别，level级别越高表示检查的payload个数就越多。 譬如我们自定义个level设计的为2，因此只有在使用在命令行使用level》2的时候，才会使用我们的payload进行检测。

#### risk字段：
风险等级，有多大几率获取破坏数据。值有1,2,3，分别表示低中高。默认的risk为1，默认检测所有风险级别的payload。 该字段影响不大。

#### clause字段：
payload在哪个语句里生效，差不多意思就是这个payload用在sql语句的哪个位置。可用的值：

    0: Always
    1: WHERE / HAVING
    2: GROUP BY
    3: ORDER BY
    4: LIMIT
    5: OFFSET
    6: TOP
    7: Table name
    8: Column name
   
我们这里测试的是order by，此处clause的字段设置为3，经过测试这里的值可以混用的，关键看sql语法。

#### where字段： 
where字段我理解的意思是，以什么样的方式将我们的payload添加进去。 1：表示将我们的payload直接添加在值得后面[此处指的应该是检测的参数的值] 如我们写的参数是id=1，设置<where>值为1的话，会出现1后面跟payload 2：表示将检测的参数的值更换为一个整数，然后将payload添加在这个整数的后面。 如我们写的参数是id=1，设置<where>值为2的话，会出现[数字]后面跟payload 3：表示将检测的参数的值直接更换成我们的payload。 如我们写的参数是id=1，设置<where>值为3的话，会出现值1直接被替换成了我们的payload。

我们的场景是order by 1 [desc]，此处我们直接将desc更换成我们的payload即可。

#### vector字段： 
vector字段表示的是payload向量，类似于一个模型的感觉。

	
    ,IF([INFERENCE],[ORIGVALUE],(select 1 from information_schema.tables))

此处为我设置的vector，INFERENCE为条件，ORIGVALUE为参数原始的值，如我传入的id=1或者desc，1和desc即为原始值，此处无所谓，在我的场景里只要为一个值即可。

request和response理解为请求的时候payload值，以及请求的值与什么样的值进行对比。 请求的payload为：

	
    ,IF([RANDNUM]=[RANDNUM],[ORIGVALUE],(select 1 from information_schema.tables))

响应的对比payload为：


    ,IF([RANDNUM]=[RANDNUM1],[ORIGVALUE],(select 1 from information_schema.tables))

大致理解就是对比if条件不等和相等，如此来进行盲注。

了解这些参数之后，接下来我们需要知道sqlmap如何将这样自定义的payload组合起来即可。于是我们跟踪一下checksqlinjection这个函数，即可知道sqlmap是如何将payload组合起来的了。

在函数中有一个重要的参数，boundary参数，这个参数是从xml目录下的boundaries.xml文件中读取出来的。每个boundary的格式如下图内容：img


其中level，clause以及where表达的意思和payload中相关标签表达的意思是一样的。 标签ptype表示参数的类型，prefix表示添加内容的前缀，suffix表示添加内容的后缀。

核心的部分是获取payloads.xml中的每一个payload，然后获取payload中的参数与boundary.xml中获取的参数进行比较。大致流程如下：

    获取payload.xml文件中的每一个payload。
    获取boundary.xml文件中的每一个boundary。
    比较判断payload中的clause是否包含在boundary的clause中，如果有就继续，如果没有就直接跳出。
    比较判断payload中的where是否包含在boundary的clause中，如果有就继续，如果没有就直接跳出。
    将prefix和suffix与payload中的request标签的内容拼接起来保存到boundpayload中。
    最后就是发送请求，然后将结果进行比较了。

Ps.因此我们在设计自定义脚本的时候需要注意的几个地方，payload中的clause标签，level标签，where标签，vector标签以及reqeust和response标签。基本上理解并设计好这些标签，就能够自定义脚本了。



### 问题一：tamper脚本是什么时候被sqlmap载入的

sqlmap的源码，大致逻辑是这样
  main()->init()->_setTamperingFunctions()
在_setTamperingFunctions函数中加载了我们配置的tamper函数。然后会把tamper函数添加到了kb.tamperFunctions里面以被后续使用。

这样看来要自定义的话这个脚本中得有个tamper函数，然后就是编写tamper函数的内


### 问题二：tamper脚本是什么时候被sqlmap调用的

tamper脚本在queryPage函数中被调用，queryPage函数是用来请求页面内容，在每次发送请求之前，先会将payload进行tamper函数处理。下图为调用between.py的脚本。


### 问题三：tamper脚本的里的内容有什么样的规范

我们随机选择一个脚本，该脚本为base64encode.py，查看脚本中的tamper内容：



可以看到内容非常简单，将payload的内容内容做了base64编码然后直接返回。Tamper有两个参数第一个参数payload即为传入的实际要操作的payload，第二个参数**kwargs为相关httpheader。譬如你想插入或则修改header的时候可以用到。

逻辑流程弄清楚之后，很容易编写自己的tamper脚本了。
