============================
Apache
============================

----------------------------------------
Apache&SSL配置
----------------------------------------
第一步，创建证书：
::

	cd d:/wamp/bin/apache/apache2.4.9/bin/
	openssl genrsa -aes256 -out ../conf/pass.key 2048
	openssl rsa -in ../conf/pass.key -out ../conf/localhost.key
	openssl req -new -x509 -nodes -sha1 -key ../conf/localhost.key -out ../conf/localhost.crt -days 999 -config ../conf/open-ssl.conf

第二步，开启相关的模块
默认写在httpd-ssl.conf上方
第三步，修改设置
SSLSessionCache        "shmcb:d:/wamp/bin/apache/apache2.4.9/conf/ssl/logs/ssl_scache(512000)"

第四步，建立虚拟站点
::

	<VirtualHost _default_:443>
	DocumentRoot "d:/open_cocoachina/cocos_portal/branches/payment_20141203"   #指定根目录
	ServerName cn.cocos.com													   #指定域名
	ServerAdmin admin@example.com											   #指定管理员
	ErrorLog "d:/wamp/bin/apache/apache2.4.9/logs/cn.cocos.com-error.log"      #指定错误日志位置
	TransferLog "d:/wamp/bin/apache/apache2.4.9/logs/cn.cocos.com-access.log"  #指定访问日志位置
	SSLEngine on
	SSLCertificateFile "d:/wamp/bin/apache/apache2.4.9/bin/localhost.crt"      #加载证书文件
	SSLCertificateKeyFile "d:/wamp/bin/apache/apache2.4.9/bin/localhost.key"   #加载秘钥文件
	<FilesMatch "\.(cgi|shtml|phtml|php)$">
	    SSLOptions +StdEnvVars
	</FilesMatch>
	<Directory "d:/open_cocoachina/cocos_portal/branches/payment_20141203">    #指定目录访问权限，注意apache2.4语法与apache2.2已经发生变化
	   SSLOptions +StdEnvVars
	   Options Indexes FollowSymLinks MultiViews
	   AllowOverride All
	   Require all granted
	</Directory>
	BrowserMatch "MSIE [2-5]" \
	         nokeepalive ssl-unclean-shutdown \
	         downgrade-1.0 force-response-1.0

	CustomLog "d:/wamp/bin/apache/apache2.4.9/logs/ssl_request.log" \
	          "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"

	</VirtualHost> 