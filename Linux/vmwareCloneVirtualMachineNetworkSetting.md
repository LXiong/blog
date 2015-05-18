# VMware 克隆虚拟机网络配置

通过 VMware 克隆虚拟机后，新虚拟机往往不能正确识别网卡，需要重新配置。

环境：

- VMware 10.0.2 build-1744117
- 虚拟机系统 Centos 6.5 64
- 网络适配器 ：NAT模式，用于共享主机的IP地址
- Win 7 64位 
- Win IP ： 192.169.10.137
- VMnet8 IP ： 169.254.206.1
- 被复制的机器的信息如下：

		[panda@slave1 ~]$ ifconfig
		eth0      Link encap:Ethernet  HWaddr 00:0C:29:D0:45:44  
		          inet addr:169.254.206.131  Bcast:169.254.206.255  Mask:255.255.255.0
		          inet6 addr: fe80::20c:29ff:fed0:4544/64 Scope:Link
		          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
		          RX packets:32 errors:0 dropped:0 overruns:0 frame:0
		          TX packets:44 errors:0 dropped:0 overruns:0 carrier:0
		          collisions:0 txqueuelen:1000 
		          RX bytes:3808 (3.7 KiB)  TX bytes:5237 (5.1 KiB)
		
		lo        Link encap:Local Loopback  
		          inet addr:127.0.0.1  Mask:255.0.0.0
		          inet6 addr: ::1/128 Scope:Host
		          UP LOOPBACK RUNNING  MTU:65536  Metric:1
		          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
		          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
		          collisions:0 txqueuelen:0 
		          RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)

通过被复制机（名字叫`slave1`）复制的复制机（准备起名叫`slave2`）是不能上网的，我们需要做一些简单的配置。

## 恢复 eth0 设备

克隆虚拟机后，`slave2`中还保留了`slave1`的MAC地址，导致网卡不能被识别。执行`ifconfig`时，找不到`eth0`设备，解决办法是将以下文件中记录的网卡信息删除，然后重启，让Linux引导时重新识别网卡。

启动虚拟机，使用 root 用户登录，执行

	# 清空下面的文件
	echo "" > /etc/udev/rules.d/70-persistent-net.rules
	# 重启slave2
	shutdown -r now

重启后，执行

	ifconfig

就可以看到 `eth0` 设备了，只是有可能还不能上网（我的`slave1`配置的是静态IP，两个机器抢一个IP肯定不能上网呀）。

## 网络设置

使用 `ifconfig` 可以很方便的对网络进行配置。

给虚拟机改主机名为 `slave2` （复制过来默认为`slave1`）

	hostname slave2

设置`eth0`的IP和子网掩码

	ifconfig eth0 169.254.206.132 netmask 255.255.255.0

添加默认路由网关

	route add default gw 169.254.206.3

## 将网络设置写入到配置文件

修改 `ifcfg-eth0`：

	vi /etc/sysconfig/network-scripts/ifcfg-eth0

配置如下:

	DEVICE=eth0	#设备名称
	TYPE=Ethernet	#网络类型
	IPADDR=169.254.206.132 #IP地址
	NETMASK=255.255.255.0	#子网掩码
	ONBOOT=yes	#开机自启
	NM_CONTROLLED=no	#可以不要
	BOOTPROTO=static	#静态IP
	GATEWAY=169.254.206.3	#默认网关

修改 `network` 文件：

	vi /etc/sysconfig/network

内容如下：

	NETWORKING=yes
	HOSTNAME=slave2
	GATEWAY=169.254.206.3

## 重启机器

	shutdown -r now

启动后，执行

	ifconfig

就可以看到已经获取到我们设置的IP地址了，执行

	ping baidu.com

发现也可以ping通了。

至此，克隆虚拟机网络配置结束，上面的配置也许不是最好的，但解决了我的问题，值得一记。

## 参考资料

- [VMware克隆虚拟机后的网络设置 ](http://blog.chinaunix.net/uid-20726500-id-4296096.html)




