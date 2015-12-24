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

 蓝牙规格：

规格名称     规格类型     分配编码     规格级别
警报类别ID     org.bluetooth.characteristic.alert_category_id     0x2A43     已采纳
警报类别ID位掩码     org.bluetooth.characteristic.alert_category_id_bit_mask     0x2A42     已采纳
警报级别     org.bluetooth.characteristic.alert_level     0x2A06     已采纳
警报通知控制点     org.bluetooth.characteristic.alert_notification_control_point     0x2A44     已采纳
警报状态     org.bluetooth.characteristic.alert_status     0x2A3F     已采纳
Appearance     org.bluetooth.characteristic.gap.appearance     0x2A01     Adopted
电池电量     org.bluetooth.characteristic.battery_level     0x2A19     已采纳
血压功能     org.bluetooth.characteristic.blood_pressure_feature     0x2A49     已采纳
血压测量     org.bluetooth.characteristic.blood_pressure_measurement     0x2A35     已采纳
人体传感器定位     org.bluetooth.characteristic.body_sensor_location     0x2A38     已采纳
引导键盘输入报告     org.bluetooth.characteristic.boot_keyboard_input_report     0x2A22     已采纳
引导键盘输出报告     org.bluetooth.characteristic.boot_keyboard_output_report     0x2A32     已采纳
引导鼠标输入报告     org.bluetooth.characteristic.boot_mouse_input_report     0x2A33     已采纳
CSC功能     org.bluetooth.characteristic.csc_feature     0x2A5C     已采纳
CSC测量     org.bluetooth.characteristic.csc_measurement     0x2A5B     已采纳
当前时间     org.bluetooth.characteristic.current_time     0x2A2B     已采纳
自行车功率控制点     bluetooth.characteristic.cycling_power_control_point     0x2A66     已采纳
自行车功率特征     org.bluteooth.characteristic.cycling_power_feature     0x2A65     已采纳
自行车功率测量     org.blueeooth.cycling_power_measurement     0x2A63     已采纳
自行车功率矢量     org.bluetooth.characteristic.cycling_power_vector     0x2A64     已采纳
日期时间     org.bluetooth.characteristic.date_time     0x2A08     已采纳
星期日期时间     org.bluetooth.characteristic.day_date_time     0x2A0A     已采纳
星期     org.bluetooth.characteristic.day_of_week     0x2A09     已采纳
Device Name     org.bluetooth.characteristic.gap.device_name     0x2A00     Adopted
日光节约时间偏移     org.bluetooth.characteristic.dst_offset     0x2A0D     已采纳
准确时间256     org.bluetooth.characteristic.exact_time_256     0x2A0C     已采纳
固件修订字符串     org.bluetooth.characteristic.firmware_revision_string     0x2A26     已采纳
血糖功能     org.bluetooth.characteristic.glucose_feature     0x2A51     已采纳
血糖测量     org.bluetooth.characteristic.glucose_measurement     0x2A18     已采纳
血糖测量环境     org.bluetooth.characteristic.glucose_measurement_context     0x2A34     已采纳
硬件修订字符串     org.bluetooth.characteristic.hardware_revision_string     0x2A27     已采纳
心率控制点     org.bluetooth.characteristic.heart_rate_control_point     0x2A39     已采纳
心率测量     org.bluetooth.characteristic.heart_rate_measurement     0x2A37     已采纳
HID控制点     org.bluetooth.characteristic.hid_control_point     0x2A4C     已采纳
HID信息     org.bluetooth.characteristic.hid_information     0x2A4A     已采纳
IEEE 11073-20601监管认证数据表     org.bluetooth.characteristic.ieee_11073-20601_regulatory_certification_data_list     0x2A2A     已采纳
中间体套囊压力     org.bluetooth.characteristic.intermediate_blood_pressure     0x2A36     已采纳
中间体温度     org.bluetooth.characteristic.intermediate_temperature     0x2A1E     已采纳
LN控制点     org.bluetooth.ln_control_point     0x2A6B     已采纳
LN功能     org.bluetooth.characteristic.ln_feature     0x2A6A     已采纳
当地时间信息     org.bluetooth.characteristic.local_time_information     0x2A0F     已采纳
定位和速度     org.bluetooth.location_and_speed     0x2A67     已采纳
制造商名称字符串     org.bluetooth.characteristic.manufacturer_name_string     0x2A29     已采纳
测量间隔     org.bluetooth.characteristic.measurement_interval     0x2A21     已采纳
型号字符串     org.bluetooth.characteristic.model_number_string     0x2A24     已采纳
导航     org.bluetooth.characteristic.navigation     0x2A68     已采纳
新警报     org.bluetooth.characteristic.new_alert     0x2A46     已采纳
Peripheral Preferred Connection Parameters     org.bluetooth.characteristic.gap.peripheral_preferred_connection_parameters     0x2A04     Adopted
Peripheral Privacy Flag     org.bluetooth.characteristic.gap.peripheral_privacy_flag     0x2A02     Adopted
PnP ID     org.bluetooth.characteristic.pnp_id     0x2A50     已采纳
定位质量     org.bluetooth.position_quality     0x2A69     已采纳
协议模式     org.bluetooth.characteristic.protocol_mode     0x2A4E     已采纳
Reconnection Address     org.bluetooth.characteristic.gap.reconnection_address     0x2A03     Adopted
记录存取控制点     org.bluetooth.characteristic.record_access_control_point     0x2A52     已采纳
参考时间信息     org.bluetooth.characteristic.reference_time_information     0x2A14     已采纳
报告     org.bluetooth.characteristic.report     0x2A4D     已采纳
报告地图     org.bluetooth.characteristic.report_map     0x2A4B     已采纳
振铃器控制点     org.bluetooth.characteristic.ringer_control_point     0x2A40     已采纳
振铃器设定     org.bluetooth.characteristic.ringer_setting     0x2A41     已采纳
RSC功能     org.bluetooth.characteristic.rsc_feature     0x2A54     已采纳
RSC测量     org.bluetooth.characteristic.rsc_measurement     0x2A53     已采纳
SC控制点     org.bluetooth.characteristic.sc_control_point     0x2A55     已采纳
扫描间隔窗口     org.bluetooth.characteristic.scan_interval_window     0x2A4F     已采纳
扫描刷新     org.bluetooth.characteristic.scan_refresh     0x2A31     已采纳
传感器定位     org.bluetooth.characteristic.sensor_location     0x2A5D     已采纳
序列号字符串     org.bluetooth.characteristic.serial_number_string     0x2A25     已采纳
Service Changed     org.bluetooth.characteristic.gatt.service_changed     0x2A05     Adopted
软件修订字符串     org.bluetooth.characteristic.software_revision_string     0x2A28     已采纳
获支持的新警报类别     org.bluetooth.characteristic.supported_new_alert_category     0x2A47     已采纳
获支持的未读警报类别     org.bluetooth.characteristic.supported_unread_alert_category     0x2A48     已采纳
系统ID     org.bluetooth.characteristic.system_id     0x2A23     已采纳
温度测量     org.bluetooth.characteristic.temperature_measurement     0x2A1C     已采纳
温度类型     org.bluetooth.characteristic.temperature_type     0x2A1D     已采纳
时间准确度     org.bluetooth.characteristic.time_accuracy     0x2A12     已采纳
时间源     org.bluetooth.characteristic.time_source     0x2A13     已采纳
时间更新控制点     org.bluetooth.characteristic.time_update_control_point     0x2A16     已采纳
时间更新状态     org.bluetooth.characteristic.time_update_state     0x2A17     已采纳
日光节约时间的时间     org.bluetooth.characteristic.time_with_dst     0x2A11     已采纳
时区     org.bluetooth.characteristic.time_zone     0x2A0E     已采纳
射频功率     org.bluetooth.characteristic.tx_power_level     0x2A07     已采纳
未读警报状态     org.bluetooth.characteristic.unread_alert_status     0x2A45     已采纳
记忆码     UUID规格     UUID     参考规格
«设备名称»     uuid16     0x2A00     蓝牙核心规格第3卷C部分第12.1节
«外观»     uuid16     0x2A01     蓝牙核心规格第3卷C部分第12.2节
«外置设备隐私标志»     uuid16     0x2A02     蓝牙核心规格第3卷C部分第12.3节
«重新连接地址»     uuid16     0x2A03     蓝牙核心规格第3卷C部分第12.4节
«外置设备首选连接参数»     uuid16     0x2A04     蓝牙核心规格第3卷C部分第12.5节
«服务更改»     uuid16     0x2A05     蓝牙核心规格第3卷G部分第7.1节
