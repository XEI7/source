title: Re重装再重装
date: 2015-11-30 17:59:04
categories: [kali]
tags: [kali,重装]
---
deb http://http.kali.org/kali sana main non-free contrib
deb-src http://http.kali.org/kali sana main non-free contrib
deb http://security.kali.org/kali-security/ sana/updates main contrib non-free
deb-src http://security.kali.org/kali-security/ sana/updates main contrib non-free
#中科大kali源
deb http://mirrors.ustc.edu.cn/kali sana main non-free contrib
deb-src http://mirrors.ustc.edu.cn/kali sana main non-free contrib
deb http://mirrors.ustc.edu.cn/kali-security/ sana/updates main contrib non-free
deb-src http://mirrors.ustc.edu.cn/kali-security/ sana/updates main contrib non-free

    :w !sudo tee %

<!--more-->


/usr/bin/unix-privesc-check

fstab
    UUID=e4419164  /code	ntfs	rw,user,noauto,exec,utf8	0	0

bash_it  candy    
    PS1="${bold_yellow}|${reset_color}${yellow}\w${reset_color}$(scm_prompt_info)${blue} →${blue} ${reset_color} ";

    仿宋ttf solarized

git clone https://github.com/sqlmapproject/sqlmap
git clone https://github.com/evilsocket/bettercap
git clone https://github.com/XEI7/Nmxwkk
git clone https://github.com/JonathanSalwan/ROPgadget
wget http://standards-oui.ieee.org/oui.txt
cat oui.txt |grep "base 16"|awk '{print $1 " : " $4}'> oui.md                   
cat oui.txt |grep "hex"|awk '{print $1 " : " $3}' >oui-md

    history  
    | awk '{CMD[$2]++;count++} END{for (a in CMD) print CMD[a]" \t " CMD[a]/count "% \t" a}' 
    | grep -v "./" | column -c3 -s " " -t |sort -nr | nl | head -n10

    tcpdump -i eth0 -tnn dst port 80 -c 1000 
    | awk -F"." '{print $1"."$2"."$3"."$4}' 
    | sort | uniq -c | sort -nr |head -20


    netstat -anlp|grep 80|grep tcp|awk '{print $5}'|awk -F: '{print $1}'
    | sort|uniq -c|sort -nr|head -n20 |
    netstat -ant 
    | awk '/:80/{split($5,ip,":");++A[ip[1]]}END{for(i in A) print A[i],i}' 
    | sort -rn|head -n20

    ps -e -o ppid,stat | grep Z | cut -d” ” -f2 | xargs kill -9
    ps -A -ostat,ppid,pid,cmd | grep -e '^[Zz]'
    sudo apt-get install fcitx fcitx-googlepinyin
    sudo apt-get install iftop pngcheck  audacity
    sudo apt-get install linux-headers-`uname -r`
    sh VMware-Workstation-Full x86.bundle


chrome
fcitx-configtool
sogou 停用 Fcitx Kimpanel组件  会使得候选字变乱码
~/.config/sublime [TAB] /Packages git clone https://github.com/xgenvn/InputHelper.git
    ^M should input like this  Ctrl+v Ctrl+m 
    dos2unix filename
    sed -i 's/^M//g' fielname
    VIM :1,$ s/^M/\r/g
    cat filename |tr -d '\r' > newfilename
```
#!/bin/sh
#/opt/node/bin/npm.ln
cd /opt/node/bin && ./npm "$@"
```
