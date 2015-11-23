title: bash常用技巧
date: 1912-12
categories: [linux]
tags: [bash,linux]
---

本文分享了在 shell 学习和使用中经常用到的一些功能和技巧。
编码规范
1 对命令的返回值进行判断 
2 临时文件采用脚本名加 PID 标识并清理
3 function 内的局部变量使用 local 限定符
4 显式函数返回 return 脚本退出 exit
5 变量名用 ${}括起来
6 命令替换使用 $() 而不是反引号
7 将变量写在脚本头或者独立成配置

参数处理
直接使用 $0,$1……，$@，$#
通过 eval 赋值
function_test key1=value1 key2=value2
在 function_test 内部使用
eval “$@”
解析参数输入, 后面就可以通过 $key1,$key2 使用了
通过 set 改变环境变量
string=“var1 var2 var3”
set -- $string
则可以通过 $1 的值为 var1，$# 的值为 3。这种方法改变了环境变量，慎重。或者在 subshell 中使用

getopts
理解 subshell/ 子进程
子进程可以继承父进程的环境变量

num=0
cat file | while read line ; do
$num++
done
echo $num
和

num=0
While read line ; do
let “num ++”
Done < file
“|”创建了一个子进程，无法将变量传给父 shell

避免常见陷阱

1. 避免 shell 参数个数限制
xargs
2. 避免 test 测试错误
[ "X$var" = Xsomething ]
3. 避免变量未初始化错误
${var:-0}
4. 避免 cd 引起路径错误
() # 或者
&& # 屏蔽
5. 更加安全的使用 $@
${1+”$@”}
6. 避免进程异常退出
trap ‘rm tempfle’ EXIT
crontab 中的元字符
%
7. 规避 xargs 的默认分割行为
find . -type f -mtime +7 -print 0 | xargs -0 rm
8. 避免拷贝错误：
cp file dir/       ## 一定记住最后的“/”
理解文件描述符
     ... >file 2>&1 # 和
     ... 2>&1 >file ## 的区别为: shell 从左到右读取参数

     ... >file 2>&1 ## 将标准输出和标准错误重定向到 file
     2>&1 >file ## 将标准输出重定向到 file，标准错误仍然为屏幕

——————-
     ... &>/dev/null ## 等价于
     ... >/dev/null 2>&1 ## 使用前者。
命令分组
(command1;command2) >log ## 子 shell 中运行命令组
{command1 ; command2 ;} >/dev/null ## 当前 shell 中运行命令组
((command1;command2)& ## 多个命令后台运行
字串替换
说明： # 前 % 后，控制字串截取方式 实例：当前目录下有如下文件

host.new offline.new online.new rd.new wugui64.new xferlog.new

需要将后缀.new 去掉

for x in \`ls *new\`; do
old_name=${x%.new}
mv $x $old_name
done
进程替换
<() 将进程的输出替换为文本做标准输入

vimdiff <() <()
同时从文件和标准输入获取：

cat file | diff – file2 ## – 代表标准输入
另外一种方式

diff <(cat file) file2
实例：diff 两台服务器的同一个配置文件

Vimdiff <(ssh server1 cat conf) <(ssh server2 cat conf)
wget 使用
不要随便修改 -t -T 选项的设置
限制使用 *，失败后返回值仍为 0
注意加 -c 和不加 -c 的程序行为
从线上下载数据要加 –limit-rate=10M
ssh 的使用
非交互使用 ssh，最好加 -n 参数

file 文件的内容为：

server1
server2
while read server ; do
ssh -n $server ‘uname -r’
done < file
远程使用 vim，加 -t 参数，分配 tty 超时，重试参数

-o ConnectTimeout=20 -o ConnectionAttempts=4
使用 rsync 前，加 –dry-run 参数 scp 加 -p 参数，保持文件时间戳一致，利用浏览器缓存

find 的使用
排除目录
find abs -path “abs/zllib” -prune -o -name “*.sh” –print
精确判断时间
touch –t time time_file
find –newer time_file
运行命令
-exec command {} \; # {}代表 find 找到的，作为 command 的参数
分离会话
1 nohup

nohup command & ## 需要注意的一点是如果 command 中包含多个命令，不要使用&& 连接，需要使用 ;
2 disown：命令敲下去发现忘记 nohup 了怎么办？使用 disown 补救 3 screen：在 wiki 中搜一下

创建安全和可维护的脚本
1 供其他进程使用的文件生成时 采用更名再 mv 的方式 如

file :>file
2 将函数和配置独立成单独的脚本

3 将不同服务器需要差异对待的变量提取成单独的配置文件

4 日志打印必须包含脚本名 basename 和时间

5 每步骤必须校验返回值

6 脚本中避免使用 *

7 保持缩进 4 个空格

8 过长的命令按照|折行

9 创建目录使用 mkdir –p

10 如果采用后台运行一定要 wait： ( command1 ; command2 ) & wait

11 对于需要获取命令输出的命令需要将 stderr 屏蔽到 /dev/null

12 抽离公共逻辑作为函数或者代码片段（导入变量）

13 保证互斥和脚本实例唯一性

参考资料
Abs — advanced bash scripting guide
unix power tools — unix 超级工具上、下
