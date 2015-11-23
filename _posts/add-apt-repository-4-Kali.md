title: add-apt-repository 4 Kali
date: 1912-12
categories: [kali]
tags: [kali,apt]
---

Kali没有了add-apt-repository 命令，要自己添加

#### 1.安装依赖软件包

> sudo apt-get install python-software-properties 
> sudo apt-get install apt-file -y
> sudo apt-file update 

#### 2.当apt-file更新完，应该能够搜索了。

> sudo apt-file search add-apt-repository 

#### 3.使用自己的add-apt-repository代码

> sudo vim /usr/sbin/add-apt-repository 

添加下列代码，并保存文件。

```
#!/bin/bash
if [ $# -eq 1 ]
NM=`uname -a && date`
NAME=`echo $NM | md5sum | cut -f1 -d" "`
then
  ppa_name=`echo "$1" | cut -d":" -f2 -s`
  if [ -z "$ppa_name" ]
  then
    echo "PPA name not found"
    echo "Utility to add PPA repositories in your debian machine"
    echo "$0 ppa:user/ppa-name"
  else
    echo "$ppa_name"
    echo "deb http://ppa.launchpad.net/$ppa_name/ubuntu oneiric main" >> /etc/apt/sources.list
    apt-get update >> /dev/null 2> /tmp/${NAME}_apt_add_key.txt
    key=`cat /tmp/${NAME}_apt_add_key.txt | cut -d":" -f6 | cut -d" " -f3`
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys $key
    rm -rf /tmp/${NAME}_apt_add_key.txt
  fi
else
  echo "Utility to add PPA repositories in your debian machine"
  echo "$0 ppa:user/ppa-name"
fi
```

 Note: In this line 
 echo "deb http://ppa.launchpad.net/$ppa_name/ubuntu oneiric main" >> /etc/apt/sources.list 
 I’ve used Oneiric. 
 You can try to use Lucid, Raring or Saucy as per your choice.


> sudo cp add-apt-repository.sh /usr/sbin/add-apt-repository
> sudo chmod o+x /usr/sbin/add-apt-repository
> sudo chown root:root /usr/sbin/add-apt-repository

#### 4.现在可以导入ppa源了

[Read More](http://www.blackmoreops.com/2014/02/21/kali-linux-add-ppa-repository-add-apt-repository/)
