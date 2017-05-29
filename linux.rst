=====================
Linux
=====================

----------------------------------------
Linux终端模式下磁盘管理
----------------------------------------

Linux终端模式下挂载U盘
========================

.. code-block:: sh

	fdisk -l

找到U盘信息例如/dev/sdc1

.. code-block:: sh
  :linenos:

	cd /mnt
	mkdir usb
	mount /dev/sdc1 /mnt/usb

----------------------------------------
Linux终端模式下加硬盘
----------------------------------------

.. code-block:: sh
  :linenos:
  
	fdisk -l #列出所有磁盘设备和其信息

	fdisk /dev/sdb #交互式设置磁盘，直接w写入分区表

	fdisk /dev/sdb #交互式分区，n创建分区，w写入更改

	mkfs.ext4 /dev/sdb1 #将磁盘格式化为ext4格式

	cd / 
	mkdir data #创建/data文件夹

	mount /dev/sdb1 /data #挂载新分区到/data文件夹

	vim /etc/fstab 
	/dev/sdb1   /data   ext4    defaults  0  0 
	设备名称    挂载点  文件系统格式     选项  dump pass

----------------------------------------
linux下配置网卡
----------------------------------------

.. code-block:: sh

	ifconfig -a                    #列出所有网卡和其配置信息
	vim /etc/network/interfaces    #打开配置文件加入配置信息

加入

.. code-block:: sh

	auto eth0
	iface eth0 inet dhcp

静态地址配置

.. code-block:: sh

	auto eth0 
	iface eth0 inet static 
	address 192.168.0.42 
	network 192.168.0.0 
	netmask 255.255.255.0 
	broadcast 192.168.0.255 
	gateway 192.168.0.1

----------------------------------------
Ubuntu与时间服务器同步
----------------------------------------
**1.** 添加时区

.. code-block:: sh

	sudo cp /usr/share/zoneinfo/Asia/ShangHai /etc/localtime 

**2.** 修改时区

.. code-block:: sh

	vim /etc/timezone

修改为Asia/ShangHai

**3.** 安装ntpdate工具

.. code-block:: sh

	sudo apt-get install ntpdate

**4.** 设置系统时间与网络同步

.. code-block:: sh

	ntpdate cn.pool.ntp.org

**5.** 将系统时间写入硬件

.. code-block:: sh

	hwclock -w

----------------------------------------
VPN PPTP设置
----------------------------------------

**1.** 安装PPTP软件： yum install pptp -y

**2.** 添加用户信息到/etc/ppp/chap-secrets:

#client                      server            secret          IP address

someone@sample.com  Sample-Vpn    123456       *

**3.** 创建配置文件到/etc/ppp/peers/文件名字叫vpn.myserver.org

vim /etc/ppp/peers/vpn.myserver.org

添加如下内容： ::

	pty "pptp 219.141.170.158 --nolaunchpppd"
	name someone@sample.com
	remotename Sample-Vpn
	require-mppe-128
	file /etc/ppp/options.pptp
	ipparam vpn.myserver.org

**4.** 注册ppp_mppe内核模块

modprobe ppp_mppe

**5.** 反注释文件/etc/ppp/options.pptp中下面内容：
::

	lock
	noauth
	refuse-pap
	refuse-eap
	refuse-chap
	nobsdcomp
	nodeflate
	require-mppe-128

**6.** 执行连接命令:

pppd call vpn.myserver.org

**7.** 验证：

ip a | grep ppp

显示的内容应该类似下面：
::

	3: ppp0:  mtu 1456 qdisc pfifo_fast state UNKNOWN qlen 3
	link/ppp
	inet 192.168.100.10 peer 192.168.100.1/32 scope global ppp0

**8.** 添加到路由

route add default dev ppp0

上面这条路由的意思是将ppp0这个虚拟网卡作为默认的网络设备，所有的流量都将从这里通过

想要控制单个的IP加入ppp0其他仍走默认网卡可以用下面的

route add 192.168.1.13 dev ppp0

**9.** 想要退出VPN连接，直接杀掉pppd进程：

killall pppd