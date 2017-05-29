===========================
Memcache
===========================

**1.** 安装依赖：

.. code-block:: sh

	yum install libevent-devel

**2.** 获取memcache安装包，并编译安装：

.. code-block:: sh

	wget http://www.memcached.org/files/memcached-1.4.24.tar.gz
	tar xvfz memcached-1.4.24.tar.gz
	cd memcached-1.4.24
	./configure && make && make test && make install

**3.** 获取memcache的PHP扩展并编译安装:

.. code-block:: sh

	wget https://pecl.php.net/get/memcache-3.0.8.tgz
	tar xvfz memcache-3.0.8.tgz
	cd memcache-3.0.8
	/usr/local/php/bin/phpize
	./configure --with-php-config=/usr/local/php/bin/php-config
	make && make install

**4.** 配置PHP配置文件，加入memcache扩展。

.. code-block:: sh

	vim /usr/local/php/etc/php.ini

加入

.. code-block:: sh

	extension="memcache.so"

验证

.. code-block:: sh

	php -m | grep memcache

结果为memcache证明安装成功。

**5.** 重启php-fpm服务。

**6.** 域名设置。

因为项目使用memcache.cocos.com作为配置，请编辑服务器的hosts文件，将memcache.cocos.com指向memcache服务器的IP地址。