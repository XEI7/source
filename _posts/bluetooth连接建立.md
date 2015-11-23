title: bluetooth连接建立
date: 1912-12
categories: [bluetooth]
tags: [bluetooth]
---

 物理信道（physical channel）是蓝牙系统的最底层结构，它以一伪随机跳频序列、特定的发送时槽定时、接入码及帧头编码来表征。蓝牙定义了一系列物理信道用于不同的应用，包括用于匹克网内设备通信的匹克网物理信道，用于查找设备的查找扫描物理信道和用于寻呼设备的寻呼扫描物理信道。两台设备必须采用相同的物理信道才能进行通信。主从设备建立连接的过程就是建立相同的匹克网信道的过程，这样主从设备才能以同样的定时和次序进行载波频率的跳变，进行数据传输，

同时可以根据匹克网接入码和帧头编码进行数据过滤和解析，避免和其他设备在同一个频段上的偶尔的相撞。


1.查询扫描过程


2.寻呼扫描过程

寻呼扫描物理信道（page scan physical channel）用于主设备寻呼从设备，是设备建立连接的必经阶段，主从设备是匹克网内设备的概念，这里用来指发起寻呼的设备和寻呼扫描设备。寻呼扫描跳频序列和寻呼请求帧的设备接入码-DAC是由从设备物理地址运算出来的，主设备以该跳频序列进行载波频率的跳变并在发送时间槽内发送寻呼请求，处于可被连接模式的从设备以固定的周期（由page scan interval决定）在一个固定的时间窗（由page scan window决定）内以某个跳频频率监听主设备的寻呼请求，监听到请求便在下个时间槽立即发送从设备寻呼响应(slave page response)，主设备在收到从设备寻呼响应的下个时间槽发送主设备寻呼响应(master page response)，该响应中包含了由主设备地址运算出来的跳频序列信息和时钟相位，从设备接收到这些信息便进入连接状态并自动成为匹克网的从设备，并再次返回从设备寻呼响应，主设备收到该响应后进入连接状态并自动成为匹克网的主设备。


应用层的连接是建立在匹克网物理信道之上的逻辑连接，主设备通过SDP查询从设备相应服务的逻辑通道号，依据该通道建立应用

层级的连接。


协议栈已经有了，使用蓝牙是非常简单的事情。

    （1）找到蓝牙设备，这是HCI层负责的，使用bluez-utils包提供的hcitool来找到蓝牙设备。

    （2）找到服务，RFCOMM是通过不同的频道（channel）来提供不同的Profile的，所以需要找到要用的服务在设备上的哪个频道上，这是通过同一个软件包里的sdptool来完成的，就是SDP，服务发现协议。

    （3）连接恰当的服务并使用。

    蓝牙的特点就是如上所述的那些了，而用户态的工具所要完成的任务就是发现服务和使用服务了。


连接的建立：

蓝牙系统有三种主要状态：待机状态，连接状态和节能状态。从待机状态向连接状态转变的过程中，有7个子状态：寻呼、寻呼扫描、查询、查询扫描、主响应、从响应、查询响应。


1．启动HCI设备
    首先，用户需要启动hcid，让HCI层的通信可以进行。对于Debian用户来说，需要安装bluez-utils包，并启动hcid。如果已经运行了 bluetooth服务，插入USB适配器后，hcid就已经在运行了，看看相关信息，见清单15.2。

清单15.2  HCI接口信息

       # hciconfig -a
        hci 0:  Type : USB
        BD Address : 11:11:11:11:11:11 ACL MTU: 678:8 SCO MTU: 48:10
        UP RUNNING PSCAN ISCAN
        RX bytes :413 acl :0 sco :0 events :19 errors :0
        TX bytes :323 acl :0 sco :0 commands :19 errors :0
        Features : 0xbf 0xfe 0x8d 0x78 0x08 0x18 0x00 0x00
        Packet type : DM1 DM3 DM5 DH1 DH3 DH5 HV1 HV2 HV3
        Link policy : RSWITCH HOLD SNIFF PARK
        Link mode : MASTER
        Name : 'inspiration -0'
        Class : 0x3e 0100
        Service Classes : Networking , Rendering , Capturing , Object Transfer , Audio
        Device Class : Computer , Uncategorized
        HCI Ver : 1.2 (0x2) HCI Rev : 0x1fe LMP Ver : 1.2 (0x2) LMP Subver : 0x1fe
        Manufacturer : Integrated System Solution Corp . (57)

    这个过程是自动的，当然也可以用hciconfig(8)来手工控制。hcid的配置文件位于/etc/bluetooth/hcid.conf， 通常使用软件包附带的版本就可以了，如果希望不用每次连接都在计算机这里确认一次PIN码的话，可以设置其中的security字段为auto，这样，每 次连接就会使用passkey设置的PIN码了。


2．寻找蓝牙设备
    HCI已经启动了，现在就可以用它来寻找蓝牙设备了，当然，一定要先开启蓝牙设备的蓝牙功能，这个不是废话，手机的蓝牙是 默认关闭的，只有在手动控制之下才会发送信号，允许被扫描到，不过设备的个体差异性太大，这里没法介绍，作者假设读者已经自己摸索或参照说明书打开了设备 的蓝牙电源。寻找蓝牙设备如清单15.3所示。

清单15.3  寻找蓝牙设备

     # hcitool scan
     Scanning ...
        00:17:00:7 B :18: B8         Motorola SLVR

之后蓝牙设备就会被顺利地找到，当然，前提是不要忘了打开手机的蓝牙开关，并允许被找，这个功能平时最好不要打开，以防在公共场合遇到安全问题。


3．查看设备提供的服务
利用SDP协议，用户还可以查看每个设备都有功能，能提供什么服务，每种基于RFCOMM的服务都使用某种协议，占据一个“频道（channel）”，这是使用蓝牙服务时的一个重要参数。

下面是例子，先看看本机，见清单15.4。

清单15.4  本机提供的蓝牙服务

        x@x :~$ sdptool browse local
        Browsing FF:FF:FF :00:00:00 ...
        Service Name : OBEX Object Push
        Service RecHandle : 0x10000
        Service Class ID List :
        " OBEX Object Push " (0x1105)
        Protocol Descriptor List :
        "L2CAP" (0x0100)
        "RFCOMM " (0x0003)
        Channel : 9
        " OBEX " (0x0008)
        Profile Descriptor List :
        " OBEX Object Push " (0x1105)
        Version : 0x0100

然后可以看看关心的设备提供的服务，手机提供的服务种类比较多，首先是SDP服务器，也就是服务发现服务器，有了这个服务，就可以接下来发现其他服务功能了，见清单15.5。

清单15.5  手机提供的SDP功能

        x@x :~$ sdptool browse 00:17:00:7 B :18: B8
        Browsing 00:17:00:7 B :18: B8 ...
        Service RecHandle : 0x0
        Service Class ID List :
        "SDP Server " (0x 1000)
        Protocol Descriptor List :
        "L2CAP" (0x 0100)
        "SDP" (0x 0001)
        Profile Descriptor List :
        "SDP Server " (0x 1000)
        Version : 0x 0100

手机的最基本功能就是用于（电话或网络）拨号，这里列出的第一项服务也是“拨号网络网关”，标识服务具体类型的字段是“Service Class ID”，它所在的频道是RFCOMM的频道1，如清单15.6所示。

清单15.6  手机提供的拨号网络功能

Service Name : Dialup Networking Gateway

    Service Description : Dialup Networking Gateway

Service Provider : Motorola

    Service RecHandle : 0x 10001

Service Class ID List :

     " Dialup Networking " (0x 1103)

Protocol Descriptor List :

     "L2CAP" (0x 0100)

       " RFCOMM " (0x 0003)

         Channel : 1

Language Base Attr List :

56    code _ ISO 639: 0x 656e

 encoding :     0x6a

58    base _ offset : 0x 100

 code _ ISO 639: 0x7a68

60    encoding :     0x6a

 base _ offset : 0xc 800

62  Profile Descriptor List :

 " Dialup Networking " (0x 1103)

64        Version : 0x 0100

除 了拨号网络服务的频道位置相对固定之外，其他服务在不同手机上的频道通常是不同的，手机一般支持的其他服务包括音频/耳机服务、车载免提服务、 OBEX对象推送服务、OBEX文件传输服务以及图片推送服务等，利用这些服务，可以利用手机拨号上网或是与手机交换图片、音乐等文件。


15.2.4  使用蓝牙
    首先介绍如何利用蓝牙取代手机数据线，以便进行拨号等工作。之后介绍利用蓝牙传送文件的几个用法，都是借助于前面提到的OBEX协议。

1．用蓝牙代替串口
    大家知道，蓝牙的一个基本功能就是模仿串口，而串口的重要作用之一（可能是最重要的了）就是拨号，传统的DTE也就是Modem。实际上，通过 RFCOMM，蓝牙连接可以反映在/dev/rfcomm0这样的字符设备上，像串口一样操作。当然，最好先定义/etc /bluetooth/ rfcomm.conf，里面根据手机的设备号和频道号写上相应设置，见清单15.7。

清单15.7  rfcomm设置

rfcomm 0 {

66          # Automatically bind the device at startup

    bind yes ；

68

        # Bluetooth address of the device

70      device 00:17:00:7 B :18: B8；

72          # RFCOMM channel for the connection

    channel 1；

74

        # Description of the connection

76      comment " Motorola SLVR L7"；

}

    这样，在启动bluetooth服务的时候，就已经自动连接上了，而不需要使用rfcomm（1）命令自己费力气了。现在，可以使用任意一个喜欢的串口程 序（minicom、gtkterm等等）来对/dev/rfcomm0进行操作了，当然也可以使用pppd建立拨号网络。


2．利用OBEX推送文件
    这是使用手机或计算机提供的“OBEX Object Push”（0x1105）服务，由另一方向其推送如文件。使用的工具是openobex-apps包里的obex test工具。首先是利用手机的Push服务向手机推送，对于本例中的手机，这是通过清单15.5所使用的sdptool命令得到的结果的一部分，清单 15.8所示。

清单15.8  手机提供的对象推送功能

78   Service Name : OBEX Object Push

Service Description : OBEX Object Push

80   Service Provider : Motorola

Service RecHandle : 0x 10008

82   Service Class ID List :

        " OBEX Object Push " (0x 1105)

84   Protocol Descriptor List :

        "L2CAP" (0x 0100)

86      " RFCOMM " (0x 0003)

       Channel : 8

88      " OBEX " (0x 0008)

可以看到，推送服务位于频道8，现在，在obex_test的命令行里指定设备和频道，如清单15.9所示。

清单15.9  向手机推送文件

x@x :~$ obex _ test -b 00:17:00:7 B :18: B8 8

90   Using Bluetooth RFCOMM transport

OBEX Interactive test client / server .

92  > c

Connect OK!

94   Version : 0x10. Flags : 0x00

> p wangxu . jpg me. jpg

96   PUT file (local , remote )> name = wangxu .jpg , size =34177

Going to send 34177 bytes

98   Made some progress ...

Made some progress ...

100  Made some progress ...

Made some progress ...

102  Made some progress ...

PUT successful !

104  > q

执行完obex-test之后，进入一个交互状态，首先建立连接，然后传送文件（本地文件名是wangxu.jpg，存在手机上叫me.jpg（这个是随意取的），最后退出。这个过程需要看着手机屏幕，可能要确认是否连接，文件存放在哪里，这个和手机有关。

现在也可以看看手机向计算机推送，首先应该让计算机进入接收状态，如果本地没有启动Object PUSH服务，可以利用sdptool把它加上，如清单15.10所示。



# bluetooth one
