title: sqlmap学习
categories: [sqlmap]
date: 1912-12
tags: [sqlmap]
---

## 首先详细看一下payload中字段：
<!--more-->
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
