title: Btproxy-Bluetooth
date: 2022-12
categories: [bluetooth]
tags: [bluetooth]
photos:
---
文章摘要
bluetooth sniffer /bluetooth pin linkkey cracker/ blueSoleil
python /usr/local/bin/bluez_simple_agent_nouser hci0  02:11:0F:19:15:57 02:11:0F:19:15:57
btproxy->mitm.py.restart_bluetoothd()->adapter.py._run(cmd)

注销掉代码mitmpy.restart_bluetoothd()
dbus.exceptions.DBusException: org.freedesktop.DBus.Error.UnknownMethod: Method "FindAdapter" with signature "s" on interface "org.bluez.Manager" doesn't exist


bluez_simple_agent_nouser  nu 110 "org.bluez.Manager"
mitm.setup_adapters <if>alreadypaired restart_bluetoothd

unning proxy on master  F1:7B:32:BA:0F:78  and slave  02:11:0F:19:15:57
Using shared adapter
Slave adapter:  hci0
Master adapter:  hci0
Looking up info on slave (04:18:0F:19:35:57)
Looking up info on master (F4:8B:32:BA:0F:78)
Spoofing master name as  Samsung Galaxy S_btproxy
Traceback (most recent call last):
  File "/usr/local/bin/bluez_simple_agent_nouser", line 4, in <module>
    __import__('pkg_resources').run_script('btproxy==0.1', 'bluez_simple_agent_nouser')
  File "/usr/lib/python2.7/dist-packages/pkg_resources.py", line 534, in run_script
    self.require(requires)[0].run_script(script_name, ns)
  File "/usr/lib/python2.7/dist-packages/pkg_resources.py", line 1438, in run_script
    execfile(script_filename, namespace, namespace)
  File "/usr/local/lib/python2.7/dist-packages/btproxy-0.1-py2.7-linux-x86_64.egg/EGG-INFO/scripts/bluez_simple_agent_nouser", line 131, in <module>
    path = manager.FindAdapter(args[0])
  File "/usr/lib/python2.7/dist-packages/dbus/proxies.py", line 70, in __call__
    return self._proxy_method(*args, **keywords)
  File "/usr/lib/python2.7/dist-packages/dbus/proxies.py", line 145, in __call__
    **keywords)
  File "/usr/lib/python2.7/dist-packages/dbus/connection.py", line 651, in call_blocking
    message, timeout)
dbus.exceptions.DBusException: org.freedesktop.DBus.Error.UnknownMethod: Method "FindAdapter" with signature "s" on interface "org.bluez.Manager" doesn't exist

Command '['python', '/usr/local/bin/bluez_simple_agent_nouser', 'hci0', '04:18:0F:19:35:57']' returned non-zero exit status 1 ['python', '/usr/local/bin/bluez_simple_agent_nouser', 'hci0', '04:18:0F:19:35:57']
python /usr/local/bin/bluez_simple_agent_nouser hci0 04:18:0F:19:35:57 failed
Trying again ..


Traceback (most recent call last):
  File "/usr/local/bin/bluez_simple_agent_nouser", line 4, in <module>
    __import__('pkg_resources').run_script('btproxy==0.1', 'bluez_simple_agent_nouser')
  File "/usr/lib/python2.7/dist-packages/pkg_resources.py", line 534, in run_script
    self.require(requires)[0].run_script(script_name, ns)
  File "/usr/lib/python2.7/dist-packages/pkg_resources.py", line 1438, in run_script
    execfile(script_filename, namespace, namespace)
  File "/usr/local/lib/python2.7/dist-packages/btproxy-0.1-py2.7-linux-x86_64.egg/EGG-INFO/scripts/bluez_simple_agent_nouser", line 131, in <module>
    path = manager.FindAdapter(args[0])
  File "/usr/lib/python2.7/dist-packages/dbus/proxies.py", line 70, in __call__
    return self._proxy_method(*args, **keywords)
  File "/usr/lib/python2.7/dist-packages/dbus/proxies.py", line 145, in __call__
    **keywords)
  File "/usr/lib/python2.7/dist-packages/dbus/connection.py", line 651, in call_blocking
    message, timeout)
dbus.exceptions.DBusException: org.freedesktop.DBus.Error.UnknownMethod: Method "FindAdapter" with signature "s" on interface "org.bluez.Manager" doesn't exist

Command '['python', '/usr/local/bin/bluez_simple_agent_nouser', 'hci0', '04:18:0F:19:35:57']' returned non-zero exit status 1 ['python', '/usr/local/bin/bluez_simple_agent_nouser', 'hci0', '04:18:0F:19:35:57']
python /usr/local/bin/bluez_simple_agent_nouser hci0 04:18:0F:19:35:57 failed
Trying again ..
 

Command '['python', '/usr/local/bin/bluez_simple_agent_nouser', 'hci0', '02:11:0F:19:15:57']' returned non-zero exit status 1 ['python', '/usr/local/bin/bluez_simple_agent_nouser', 'hci0', '02:11:0F:19:15:57']
paired
Moved system SDP PSM 1 to 101 so its up for grabs.
Blocked rfcomm channel 2
libudev: udev_monitor_enable_receiving: bind failed: Resource temporarily unavailable
Blocked l2cap psm 31
Blocked l2cap psm 17
Blocked l2cap psm 15


主/从设备都会使用蓝牙协议，直到主设备发出请求，从属设备通常保持待机状态。并且主设备可以连接到多台设备，而从属设备仅仅智能连接一台设备。
代理会搜寻设备名称和类，所以其可以将设备名称和类复制到使用的蓝牙适配器中。在本例中，因为只有一个蓝牙适配器，所以只会复制从属设备的属性。

btproxy F4:8B:32:BA:0F:78 04:18:0F:19:35:57

<!--more-->
```bash
sudo apt-get install bluez bluez-utils bluez-tools libbluetooth-dev python-dev


bluez的编译安装依赖好些软件，下面记录下，可能比较简陋。
configure: error: GLib >= 2.28 is required
解决方法：
一般glib会被安装，主要是一些开发文件，如头文件被安装，ubuntu如下解决：
     sudo apt-get install  libglib2.0-dev

     configure: error: D-Bus >= 1.6 is required
     apt-get install DBus
     sudo apt-get install libudev-dev
     sudo apt-get install libical-dev

     configure: error: readline header files are required
     解决方法：
      sudo apt-get install libreadline-dev
     checking systemd system unit dir... configure: error: systemd system unit directory is required
     checking systemd user unit dir... configure: error: systemd user unit directory is required
     解决方法：
     ./configure  --disable-systemd
     或者
     ./configure  --with-systemdsystemunitdir=/lib/systemd/system --with-systemduserunitdir=/usr/lib/systemd



```bash
btproxy <master-bt-mac-address> <slave-bt-mac-address>
```
The master 是发送连接请求方
the slave  是服务监听方

hcitool dev 查看本地
hcitool scan 扫描
hcitool inq  查询
hcitool cc  创建连接

hciconfig hci0 up
 > Can't init device hci0: Operation not possible due to RF-kill (132)
rfkill list bluetooth
rfkill unlock bluetooth



hcidump -at


