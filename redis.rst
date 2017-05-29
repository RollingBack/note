=====================
Redis
=====================

----------------
redis安装
----------------

第一步.下载最新版redis
====================================
官方网站：

http://redis.io/

https://raw.github.com/ServiceStack/redis-windows/master/downloads/redis64-latest.zip

http://download.redis.io/releases/redis-3.0.2.tar.gz


第二步.解压到文件夹并安装
====================================
windows: ::

	redis-server.exe --service-install redis.windows.conf --loglevel verbose

linux: ::

	redis-server /etc/redis.conf &

第三步.测试
====================================

$ redis-cli

打开命令提示：::

	redis 127.0.0.1:6379>

输入PING命令，如果回应PONG，则安装成功 ::

	redis 127.0.0.1:6379> ping

	PONG

-----------------------------------------
redis笔记
-----------------------------------------

**1.** 100万的数据大概占用100MB的内存空间，未压缩情况下。

**2.** 如果只是拿来做缓存，记得防御编程，不要让redis失效了就整站挂掉。

.. code-block:: php

	<?php	
	if (!function_exists('get_redis')){
	    /**
	     * @param bool $close
	     *
	     * @return bool|Redis
	     */
	    function get_redis($close = false){
	        try{
	            static $_redis = null;
	            if($close){
	                if($_redis){
	                    $_redis->close();
	                }
	                return true;
	            }
	            if($_redis == null){
	                $ci = & get_instance();
	                $redis_config = $ci->config->item('redis');
	                if(empty($redis_config)){
	                    $redis_config['host'] = 'redis.cocos.com';
	                    $redis_config['port'] = 6379;
	                }
	                if(class_exists('Redis')){
	                    $_redis = new Redis();
	                }else{
	                    throw new Exception('Redis php client invalid');
	                }
	                if (!($_redis->pconnect( $redis_config['host'], $redis_config['port'], 2))){
	                    throw new Exception('Redis server invalid');
	                }
	            }
	            return $_redis;
	        }catch (Exception $e){
	//            cocos_log('Redis Error:'.$e->getMessage());
	            return false;
	        }
	    }
	}

	if(!function_exists('fetch_in_redis')){
	    /**
	     * 获取redis缓存的入口，防止redis有问题而页面500
	     * @param $key
	     *
	     * @return bool|string
	     */
	    function fetch_in_redis($key){
	        $redis = get_redis();
	        if($redis){
	            $result = $redis->get($key);
	            return $result;
	        }
	        return false;
	    }
	}

	if(!function_exists('cache_in_redis')){
	    /**
	     * redis缓存入口，防止redis有问题而页面500
	     * @param $key
	     * @param $value
	     * @param $ttl
	     *
	     * @return bool
	     */
	    function cache_in_redis($key, $value, $ttl=''){
	        $redis = get_redis();
	        if(!$ttl){
	            $ci = & get_instance();
	            $ttl = $ci->config->item('array_cache_time');
	            $ttl = $ttl?$ttl:3600;
	        }
	        if($redis && $redis->set($key, $value)){
	            if($ttl>time()){
	                $redis->expireAt($key, $ttl);
	            }else{
	                $redis->expire($key, $ttl);
	            }
	            return true;
	        }
	        return false;
	    }
	}

**3.** 和memcache的区别有很多，最大的区别（对于我来说）是redis内置了很多实用的数据结构。但如果只是做字符串缓存，
二者使用上没有太大区别，但是如果你是用作页面缓存，服务器用nginx，nginx可以直接从memcache中获取数据（使用相应的模块）。

**4.** 使用widnows的同学需要注意了，下面的有个选项需要你注意：

::

	# The heap memory mapped file must reside on a local path for heap sharing 
	# between processes to work. A UNC path will not suffice here. For maximum 
	# performance this should be located on the fastest local drive available.
	# This value defaults to the local application data folder(e.g.,
	# "%USERPROFILE%\AppData\Local"). Since this file can be very large, you
	# may wish to place this on a drive other than the one the operating system  
	# is installed on.
	# 选个比较空的磁盘新建个文件夹，并指向它，因为这个文件夹将来会很大。。。
	# Note that you must specify a directory here, not a file name.
	heapdir <directory path(absolute or relative)>

最近redis也有了相应的模块，还和lua成了好基友 `(openresty) <http://openresty.org/>`_。