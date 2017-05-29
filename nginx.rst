====================================
Nginx
====================================

------------------------------------
LNMP环境包安装配置
------------------------------------

**1.** 安装虚拟机，系统为CentOS

**2.** 下载并安装LNMP一键安装包：

首先需要获取ROOT权限之后在终端中运行
::

	wget -c http://soft.vpser.net/lnmp/lnmp1.1-full.tar.gz && tar zxf lnmp1.1-full.tar.gz && cd lnmp1.1-full && ./centos.sh

**3.** 安装subversion
::

	yum install subversion

**4.** cd /data转到要检出项目的目录中，执行下面的命令：
::

	svn checkout https://svn.sample/svn/open/ --username=your_username

提示输入密码，输入密码回车，会询问是否保存认证等，都回答yes并回车

**5.** cd /root/lnmp1.1-full/  转到你释放并安装lnmp环境的目录下
::

	./vhost.sh

运行vhost.sh来生产虚拟站点
::

	Please input domain：
	(Default domain：www.lnmp.org):sample.com
	输入要生成虚拟站点的域名
	Do you want to add more domain name?(y/n)
	输入n,拒绝添加其他的域名到该虚拟站点
	Please input the directory for the domain:sample.com:
	(Default directory: /home/wwwroot/sample.com):
	/data/open/
	输入svn检出的项目中对应的文件夹位置
	Allow Rewrite rule?(y/n)
	n
	是否写入伪静态规则，回答否（n）
	Allow access_log(y/n)
	y
	是否为这个虚拟站点开启访问日志，回答是（y）
	Type access_log name(Default access log file:sample.com.log):
	输入访问日志名称，默认sample.com.log,直接回车使用默认即可,日志默认位置是/home/wwwlogs/
	Press any key to start create virtual host...

在配置虚拟站点时需要加入CI的重写规则

.. code-block:: php

	location / {
			if (!-e $request_filename) {
				rewrite ^/(.*)$ /index.php?$1  last;
			}
	}

使用OpenSSL生成证书和密钥文件，问及密码时注意统一，并需要记住，重启nginx服务时需要输入密码
::

	openssl genrsa -des3 -out sample.com.key 2048  
	openssl rsa -in sample.com.key -out sample.com.key(无密码版本)
	openssl req -new -x509 -key sample.com.key -out sample.com.crt -days 3650 
	openssl req -new -key sample.com.key -out sample.com.csr 
	openssl x509 -req -days 3650 -in sample.com.csr -CA sample.com.crt -CAkey sample.com.key -CAcreateserial -out sample.com.crt
	cat sample.com.key sample.com.crt > sample.com.pem

使用SSL协议的网站配置
在非SSL协议的基础上，监听端口从80改为443
server里加入ssl协议语句
::

	ssl                  on;
	ssl_certificate      ssl/sample.com.crt;
	ssl_certificate_key  ssl/sample.com.key;
	ssl_session_timeout  5m;
	ssl_protocols  SSLv3 TLSv1;
	ssl_ciphers  HIGH:!ADH:!EXPORT56:RC4+RSA:+MEDIUM;
	ssl_prefer_server_ciphers   on;

sample.com需要把默认的80端口下的server_name和root都指向https://sample.com不然很多css和图片没法访问
::

	server{
		listen 80;
		server_name sample.com;
		rewrite ^(.*) https://$server_name$1 permanent;
	}

如果需要在虚拟机里访问相应的网站
vim /etc/hosts 打开hosts文件并加入
**1.** 7.0.0.1 sample.com

如果需要在虚拟机外访问，则需要在主机的hosts文件中加入虚拟机的IP（ifconfig可以查看到）

外部访问需要在防火墙中开放相应端口
打开防火墙配置文件
vim /etc/sysconfig/iptables
加入对22、80、443这三个端口的开放 ::

	-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
	-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
	-A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT

重启防火墙 ::

	/etc/init.d/iptables restart

关闭默认的桌面环境
vim /etc/inittab
最下面一句将数字5改为3

.. code-block:: sh

	id:5:initdefault:

重启电脑
通过putty连接进行管理

---------------------------------
Nginx 502错误解决方案
---------------------------------

原因1：
虽然nginx 502 Bad Gateway不一定是这个引起的，但是如果一个测试环境没负载的情况下发生502，
十有八九是这个原因引起的。这个问题的根源不是nginx本身，而是nginx告诉你后端的php-fpm或者
uwsgi没有正常工作。对于php-fpm，遇见过两回，问题竟然是这个：
::

	location ~ [^/]\.php(/|$)
	{
		# comment try_files $uri =404; to enable pathinfo
		try_files $uri =404;
		fastcgi_pass  unix:/tmp/php-cgi.sock;
		fastcgi_index index.php;
		include fastcgi.conf;
		#include pathinfo.conf;
	}

	location ~ [^/]\.php(/|$)
	{
		# comment try_files $uri =404; to enable pathinfo
		try_files $uri =404;
		fastcgi_pass  127.0.0.1:9000;
		fastcgi_index index.php;
		include fastcgi.conf;
		#include pathinfo.conf;
	}

上面的是 nginx Server里的一段配置，上面两段的区别就在于fastcgi_pass后面的后端协议不同，
一个是unix的套接字协议，一个是tcp协议。这个要与php-fpm的配置应该一致。

原因2：
后端超时,nginx没有从后端获取到任何数据。以php-fpm为例：

::
	
	; The timeout for serving a single request after which the worker process will
	; be killed. This option should be used when the 'max_execution_time' ini option
	; does not stop script execution for some reason. A value of '0' means 'off'.
	; Available units: s(econds)(default), m(inutes), h(ours), or d(ays)
	; Default Value: 0
	;这个选项只是php请求数据库超时后常见的一个解决方法，并不适用所有情况，给个合适的值。
	request_terminate_timeout = 0


---------------------------------
Nginx 重定向带POST参数
---------------------------------

::

	return 307 http://www.sample.com$request_uri;

---------------------------------
Nginx404页面重定向
---------------------------------

error_page 404 https://sample.cocos.com/404.html;


----------------------------------
Nginx重写，去掉index.php
----------------------------------

.. code-block:: sh

	location / {
		if(!-e $request_filename) {
			rewrite ^/(.*)$ /index.php?$1 last;
		}
	}