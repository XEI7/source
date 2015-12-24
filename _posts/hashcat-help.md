title: hashcat-help
date: 2015-12-05 18:49:12
categories: [安全]
tags: [kali]
---
HashCat主要分为三个版本：Hashcat、oclHashcat-plus、oclHashcat-lite。
这三个版本的主要区别是：HashCat只支持CPU破解。
oclHashcat-plus支持使用GPU破解多个HASH，并且支持的算法高达77种。
oclHashcat-lite只支持使用GPU对单个HASH进行破解，支持的HASH种类仅有32种，但是对算法进行了优化，可以达到GPU破解的最高速度。如果只有单个密文进行破解的话，推荐使用oclHashCat-lite。
<!--more-->
## General:
    -m,  --hash-type=NUM               Hash-type, see references below
    -a,  --attack-mode=NUM             Attack-mode, see references below
  Misc:

    --hex-salt                    Assume salt is given in hex
    --hex-charset                 Assume charset is given in hex
    --runtime=NUM                 Abort session after NUM seconds of runtime
## * Attack modes:
    0 = Straight
    1 = Combination
    2 = Toggle-Case
    3 = Brute-force
    4 = Permutation
    5 = Table-Lookup
    6 = Prince

## Built-in charsets:

    ?l = abcdefghijklmnopqrstuvwxyz
    ?u = ABCDEFGHIJKLMNOPQRSTUVWXYZ
    ?d = 0123456789
    ?s =  !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
    ?a = ?l?u?d?s
    ?b = 0x00 - 0xff
-

    ?l表示a-z，?u表示A-Z，?d表示0-9，?a表示键盘上所有的特殊字符，
    ?s表示键盘上所有的可见字符，?h表示8bit 0xc0-0xff的十六进制，
    ?D表示8bit的德语字符，?F表示8bit的法语字符，?R表示8bit的俄语字符。
 
## Custom charsets:

    -1,  --custom-charset1    
    -2,  --custom-charset2   
    -3,  --custom-charset3    
    -4,  --custom-charset4    
--custom-charset1=?dabcdef 
: sets charset ?1 to 0123456789abcdef  
-2 mycharset.hcchr 
: sets charset 
?2 to chars contained in file
来表示，在掩码中用?1、?2、?3、?4来表示。

比如说我要设置自定义字符集1为小写+数字，那么就加上
-- custom-charset1 ?l?d
如果要设置自定义字符集2为abcd1234，那么就加上
--custom-charset2 abcd1234
如果要破解8位的小写+数字，那么需要设置自定义字符集1为
--custom-charset1 ?l?d
设置掩码为?1?1?1?1?1?1?1?1。 如果已知密码的第一位为数字，长度为8位，后几位为大写+小写，那么需要设置自定义字符集1为
--custom-charset1 ?l?u
设置掩码为?d?1?1?1?1?1?1?1。

{掩码的长度}

对于已知长度的密码，可以使用固定长度的掩码进行破解。比如要破解11位数字，就可以这样写掩码?d?d?d?d?d?d?d?d?d?d?d。

对于想要破解一些未知长度的密码，希望软件在一定长度范围内进行尝试的，可以使用--increment参数，并且使用--increment-min ?定义最短长度，使用--increment-max ?定义最大长度。比如要尝试6-8位小写字母，可以这样写

--increment --increment-min 6 --increment-max 8 ?l?l?l?l?l?l?l?l
{举例}

破解8-11位数字+小写
oclHashcat-plus64.exe --hash-type 100 --attack-mode 3 --increment --increment-min 8 --increment-max 11 --custom-charset1 ?l?d d:sha1.txt ?1?1?1?1?1?1?1?1?1?1?1 


## Resources:

    -c,  --segment-size=NUM            Size in MB to cache from the wordfile
    -n,  --threads=NUM                 Number of threads
    -s,  --words-skip=NUM              Skip number of words (for resume)
    -l,  --words-limit=NUM             Limit number of words (for distributed)

## * Match Hash types:

     0 = MD5
    10 = md5($pass.$salt)
    20 = md5($salt.$pass)
    30 = md5(unicode($pass).$salt)
    40 = md5($salt.unicode($pass))
    50 = HMAC-MD5 (key = $pass)
    60 = HMAC-MD5 (key = $salt)
    100 = SHA1
    110 = sha1($pass.$salt)
    120 = sha1($salt.$pass)
    130 = sha1(unicode($pass).$salt)
    140 = sha1($salt.unicode($pass))
    150 = HMAC-SHA1 (key = $pass)
    160 = HMAC-SHA1 (key = $salt)
    200 = MySQL323
    300 = MySQL4.1/MySQL5
    400 = phpass, MD5(Wordpress), MD5(phpBB3), MD5(Joomla)
    500 = md5crypt, MD5(Unix), FreeBSD MD5, Cisco-IOS MD5
    900 = MD4
    1000 = NTLM
    1100 = Domain Cached Credentials, mscash
    1400 = SHA256
    1410 = sha256($pass.$salt)
    1420 = sha256($salt.$pass)
    1430 = sha256(unicode($pass).$salt)
    1440 = sha256($salt.unicode($pass))
    1450 = HMAC-SHA256 (key = $pass)
    1460 = HMAC-SHA256 (key = $salt)
    1600 = md5apr1, MD5(APR), Apache MD5
    1700 = SHA512
    1710 = sha512($pass.$salt)
    1720 = sha512($salt.$pass)
    1730 = sha512(unicode($pass).$salt)
    1740 = sha512($salt.unicode($pass))
    1750 = HMAC-SHA512 (key = $pass)
    1760 = HMAC-SHA512 (key = $salt)
    1800 = SHA-512(Unix)
    2400 = Cisco-PIX MD5
    2410 = Cisco-ASA MD5
    2500 = WPA/WPA2
    2600 = Double MD5
    3200 = bcrypt, Blowfish(OpenBSD)
    3300 = MD5(Sun)
    3500 = md5(md5(md5($pass)))
    3610 = md5(md5($salt).$pass)
    3710 = md5($salt.md5($pass))
    3720 = md5($pass.md5($salt))
    3810 = md5($salt.$pass.$salt)
    3910 = md5(md5($pass).md5($salt))
    4010 = md5($salt.md5($salt.$pass))
    4110 = md5($salt.md5($pass.$salt))
    4210 = md5($username.0.$pass)
    4300 = md5(strtoupper(md5($pass)))  
    4400 = md5(sha1($pass))
    4500 = Double SHA1
    4600 = sha1(sha1(sha1($pass)))
    4700 = sha1(md5($pass))
    4710 = sha1($salt.$pass.$salt)
    4800 = MD5(Chap), iSCSI CHAP authentication
    5000 = SHA-3(Keccak)
    5100 = Half MD5
    5200 = Password Safe SHA-256  
    5300 = IKE-PSK MD5  
    5400 = IKE-PSK SHA1
    5500 = NetNTLMv1-VANILLA / NetNTLMv1-ESS
    5600 = NetNTLMv2
    5700 = Cisco-IOS SHA256
    5800 = Android PIN
    6300 = AIX {smd5}
    6400 = AIX {ssha256}
    6500 = AIX {ssha512}
    6700 = AIX {ssha1}
    6900 = GOST, GOST R 34.11-94
    7000 = Fortigate (FortiOS)
    7100 = OS X v10.8 / v10.9
    7200 = GRUB 2
    7300 = IPMI2 RAKP HMAC-SHA1
    7400 = sha256crypt, SHA256(Unix)
    7900 = Drupal7
    8400 = WBB3, Woltlab Burning Board 3
    8900 = scrypt
    9200 = Cisco $8$
    9300 = Cisco $9$
    9800 = Radmin2
    10000 = Django (PBKDF2-SHA256)
    10200 = Cram MD5
    10300 = SAP CODVN H (PWDSALTEDHASH) iSSHA-1
    99999 = Plaintext

  Specific hash types:

    11 = Joomla < 2.5.18
    12 = PostgreSQL
    21 = osCommerce, xt:Commerce
    23 = Skype
    101 = nsldap, SHA-1(Base64), Netscape LDAP SHA
    111 = nsldaps, SSHA-1(Base64), Netscape LDAP SSHA
    112 = Oracle 11g/12c
    121 = SMF > v1.1
    122 = OS X v10.4, v10.5, v10.6
    123 = EPi
    124 = Django (SHA-1)
    131 = MSSQL(2000)
    132 = MSSQL(2005)
    133 = PeopleSoft
    141 = EPiServer 6.x < v4
    1421 = hMailServer
    1441 = EPiServer 6.x > v4
    1711 = SSHA-512(Base64), LDAP {SSHA512}
    1722 = OS X v10.7
    1731 = MSSQL(2012 & 2014)
    2611 = vBulletin < v3.8.5
    2612 = PHPS
    2711 = vBulletin > v3.8.5
    2811 = IPB2+, MyBB1.2+
    3711 = Mediawiki B type
    3721 = WebEdition CMS
    7600 = Redmine Project Management Web App
