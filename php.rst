=====================
PHP
=====================

----------------
php100个坑
----------------

1. array_merge里的坑
================================================

array_merge使用时最好把参数全部加上(array)强制转换为array，否则，如果参数里有空或者空字符，
你可能得不到你想要的结果。


2. 脚本太长导致变量被替换。
====================================================

.. code-block:: PHP
	:linenos:

	$data = $this->db->get()->result_array();

大家都喜欢$data, 或者你在维护别人的代码，免不了在中间加代码对数据进行处理。如果脚本有个几百行，
你可能轻易发现不了，你现在用的变量上面已经声明并且赋值了。还有尽量不要让你的代码超过几百行，用函数或者
方法隔离一下，或者使用命名空间。

3. 反复读数据库导致效率低。
======================================================
有些程序员很喜欢把代码封装成面向对象的，但是请控制取获取数据的次数，一次获取反复使用。

4. readfile导致网站流量暴增。
=======================================================
用readfile可以获取网络文件，如果在服务器上通过这种方法获取其他服务器上的文件，请记住，你在用你公司
的服务器当'路由器'，如果你很走运，公司的运维人员可能会找到你让你改代码，如果不走运，你有可能因为带
宽问题让公司和你自己蒙受损失。

5. 删除数据的sql用子查询还是表连接，什么区别。
===================================================================================
用子查询能保证唯一性，而表连接不能，但表连接速度更快。

6. 用phpexcel导出大文件无法导出。
===================================================================
用CSV吧，我们曾经用CSV导出过几百万的数据，大小在几百兆左右，如果你的电脑不够好不要试图用编辑器打开它。
你会后悔的。

7. ob_start是什么？
===========================================================
OB缓存，PHP将输出缓存并不立即输出，你可以用ob_flush将缓存输出，或者用ob_get_content将缓存保存进入变量。
如果你不知道这东西怎么用，尽量不要使用，如果你发现输出的图像文件一直报错，试试ob_end_clean把输出缓存清
除。

8. csv文件里有多余的逗号怎么办？
==========================================================
用全角逗号，而半角逗号只用于分隔。

9. 来了新需求又要加字段，可我的表已经有50个字段了。
=================================================================================
拆掉。

10. utf-8的BOM头带来的麻烦。
===============================================================
使用UTF-8编码的php文件中如果含有BOM头，有可能导致网页显示异常。如果想用编辑器编码，请设置好编辑器的编码和缩进。经常
有人在UTF-8的项目里放GBK的编码的源文件。运行不会有问题，但别人没法看你的注释了，被别人的编辑器编辑过后，很有可能永远
成为乱码。

11. '\n',"\n"是一样的吗？
============================================================
很简单但是很容易被忽视。

12. 不要往从库里写数据。
=========================================================
使用主从库同步的情况下请不要往从库里插数据或者执行更新操作。如果没办法测试，比如修复线上的问题，用主库吧。
在从库配置文件中加入read_only=1, 建立普通账号，进行开发。

13. 前后台的数据交互必须要写ajax接口吗？
===================================================================
如果不需要实时获取数据，并且数据量很大可以这么做：

.. code-block:: html
	
	<script type="text/javascript" src="sample.php"></script>

sample.php

.. code-block:: php
	:linenos:

	var data = 
	<?php
		$data = array(
			0=>'a',
			1=>'b',
			'name' => 'john'
		);
		echo json_encode($data);

14. 我声明的常量没起作用。
=============================================================
毫无疑问，它已经被声明过了。

15. 我的代码在本地一直无法执行，源代码都暴露出来，但是在测试机或者同事的电脑上是好的。
===============================================================================================================
你没开短标签，而别人开了就会导致这样的问题。

16. 把文件暴露在网站根目录下面。
================================================================
不要把重要的文件放到网站根目录下面，它有可能会泄露你的信息。事实上，PHP脚本都可以不放在网站根目录下面，网站也能运行。
正确书写服务器配置文件，给每个文件夹放个index.html是个不错的办法。不要用SVN直接检出项目到网站根目录，因为SVN会在根目
录下创建额外的隐藏文件。

17. PHP函数多，记不住。
===================================================
不应该要求全记住，因为实在太多，而且PHP的函数不太一致。你需要PHPStorm这种IDE而不是某个简单的文字编辑器。我在做维护代码
的工作的时候发现，PHPStorm能够发现你不容易发现的错误，能极大的提升你的编码和纠错效率。比如你上面声明了个变量$whatalongname，
下面却写成$wahtalongname，PHPStorm会提示你$whatalongname，声明了但是没有被使用。嵌套级别比较多的时候，你的变量声明在
了错误的级别，获取不到这个变量，PHPStorm下面会提示你没有声明变量，这种错误很难发现，因为使用一个没有赋值的变量不会引起
什么error。总能在用某个用文字编辑器的大牛的代码里发现很低级的错误。

18. 时区设置。
===========================================
获取了一样的时间戳，总是差几个小时，这时候该看看时区设置了，php.ini里有，同时代码里也可以设置。

19. CI 框架爆出错误，但是看不见错误信息，而是提示找不到语言文件。
==============================================================================
你把CI的语言配置为了其他语言，而不是english，把语言设置改为english就可以看见报错了。这种报错基本都是SQL错误。我还没看到过其他错误引起这个问题。

20.static变量误用
============================================================
在一个脚本里，不能用static变量写调用超过两次的函数，因为结果会乱掉。第一次的结果对第二次产生了影响。下面的函数
之前有人把$sons声明为静态变量，如果脚本第二次调用就乱了。

.. code-block:: php

	<?php 
		//从一个多维数组里寻找子节点，使用了递归和引用符号
		function get_my_son($array, &$sons=array()){
	        if(count(current($array))==1){
	            return ;
	        }
	        foreach($array as $val){
	            if($val['id']>0){
	                $sons[] = $val['id'];
	                $this->get_my_son($val['sub_category'], $sons);
	            }
	        }
	        return $sons;
	    }
	?>

21.ZIP解包
=================================================================
需求需要解包Zip，用PHP的ZipArchive类来做，当时就担心会有中文文件夹或者文件会乱码。果不其然。

.. code-block:: php
	
	<?php
	$arc = new ZipArchive();
    $arc->open($path);
    for ($i=0; $i<$arc->numFiles;$i++) {
        $name = $arc->getNameIndex($i);
        $encode = mb_detect_encoding($name);
        if($encode!='ASCII'){
            echo mb_convert_encoding($name, 'UTF-8', 'CP936');
        }
    }

怎么搞都是乱码，找了半天，找到两个解决的办法：

**1.** 用unzip -O CP936 解包, 但是似乎并没有这样的选项。只能作罢。

**2.** 用Python。感谢在博客上放代码的仁兄，解压后没了乱码，代码相当简洁。

.. code-block:: python
	
	#!/usr/bin/env python
	# -*- coding: utf-8 -*-
	# uzip.py

	import os
	import sys
	import zipfile

	print "Processing File " + sys.argv[1]

	file=zipfile.ZipFile(sys.argv[1],"r");
	for name in file.namelist():
	    utf8name=name.decode('gbk')
	    print "Extracting " + utf8name
	    pathname = os.path.dirname(utf8name)
	    if not os.path.exists(pathname) and pathname!= "":
	        os.makedirs(pathname)
	    data = file.read(name)
	    if not os.path.exists(utf8name):
	        fo = open(utf8name, "w")
	        fo.write(data)
	        fo.close
	file.close()

22.json_decode()默认返回的是对象。
=======================================================
json_decode()能够将json字符串转换为对象或者数组，关键的参数是第二个参数， 默认第二个参数为false,也即转换为对象。想转换为数组，请将第二个参数设为true

23.子类和父类具有同名的方法。
==================================================
父类的方法会被覆盖。

.. code-block::php

	<?php

	class A{
			private function test(){
					echo 1;
			}
	}

	class B extends A{
			public function test(){
					echo 2;
			}
	}

	$b = new B();
	$b->test();

	?>

上面代码运行的结果是2。

------------------------
有用的代码段
------------------------

**1.** 合并数组，键相同的合并

.. code-block:: php 

	<?php
	    //合并数组，键相同的合并
	    function array_combine_($keys,$values){
	        $result = array();
	        foreach ($values as $i => $val) {
	            $result[$keys[$i]][] = $val;
	        }
	        array_walk($result, create_function("&$v", "$v = (count($v)==1)?array_pop($v):$v;"));
	        return $result;
	    }
	?>

**2.** PDO的使用,最基本的东西都在了。

.. code-block:: php

	<?php 
	
		$dbh = new PDO("mysql:host=$host;dbname=$dbname",$user,$password);
		$dbh->setAttribute(PDO::ATTR_ERRMODE,PDO::ERRMODE_WARNING);
		//执行一条SQL语句，返回结果集作为PDOStatement对象
		$rs = $dbh->query($sql);
		$rs->setFetchMode(PDO::FETCH_NUM);
		$rs->fetch();
		$rs->fetchAll();
		PDO::FETCH_NUM
		PDO::FETCH_ASSOC
		PDO::FETCH_BOTH
		//执行预拼装的SQL，支持?和:filedname
		$change = $dbh->prepare($sql);
		$change->execute();
		//获取最后插入的ID号
		$dbh->lastInsertId();
		//执行SQL，返回受影响行数
		$dbh->exec();

**3.** 一个将数据库的数据构造成树状结构数组的函数，使用了匿名函数和递归。

.. code-block:: php

	<?php

	function get_cat_tree($id=0){		
	        $categories = fetch_in_redis('COCOS_WEB_STORE_CATEGORY_TREE_'.$id);
	        if(!$categories){
	            $all_category = $this->get_all_cat();
	            $categories = array();
	            $get_son = function($id,$all_category) use (&$get_son){
	                $re = array();
	                foreach($all_category as $key=>$val){
	                    if($val['fid']==$id){
	                        $re[$val['id']] = $val;
	                        unset($all_category[$key]);
	                        $re[$val['id']]['sub_category'] = call_user_func($get_son, $val['id'], $all_category);
	                    }
	                }
	                if(count($re)>0){
	                    return $re;
	                }
	                return ;
	            };
	            foreach($all_category as $key=>$val){
	                if($val['fid']==$id){
	                    $categories[$val['id']] = $val;
	                    unset($all_category[$key]);
	                    $categories[$val['id']]['sub_category'] = call_user_func($get_son, $val['id'], $all_category);
	                }
	            }
	            cache_in_redis('COCOS_WEB_STORE_CATEGORY_TREE_'.$id, serialize($categories));
	        }else{
	            $categories = unserialize($categories);
	        }
	        return $categories;
	    }

**4.** 格式化文件大小的函数

.. code-block:: php

	<?php
	if(!function_exists('format_file_size')){
	    /**
	     * @param $size
	     *格式化文件大小显示，没指名单位时为MB
	     * @return string
	     */
	    function format_file_size($size){
	        $ext = strtolower(substr($size, -2, 2));
	        $float = substr($size, 0, -2);
	        switch($ext){
	            case 'mb':
	                $file_size = bcmul($float, 1024, 2);
	                break;
	            case 'kb':
	                $file_size = $float;
	                break;
	            case 'gb':
	                $file_size = bcmul($float, 1024*1024, 2);
	                break;
	            default:
	                $file_size = bcmul($size, 1024, 2);
	                break;
	        }
	        if($file_size == 0){
	            return 0;
	        }elseif($file_size < 1 && $file_size > 0){
	            return round(bcmul($file_size, 1024, 2), 1).'Byte';
	        }elseif($file_size < 1024 && $file_size > 1 ){
	            return round(bcdiv($file_size, 1, 2), 1).'KB';
	        }elseif($file_size >= 1024 && $file_size < 1048576  ){
	            return round(bcdiv($file_size, 1024, 2), 1).'MB';
	        }else{
	            return round(bcdiv($file_size, 1048576, 2), 1).'GB';
	        }
	        return 0;
	    }
	}

**5.** 转换阿拉伯数字为中文数字的函数

.. code-block:: php

	<?php 
	if(!function_exists('to_chinese_num')){

	    /**
	     * @param $num
	     *
	     * @return string
	     */
	    function to_chinese_num($num){
	        $chinese_num = ['零', '一', '二', '三', '四', '五', '六', '七', '八', '九' ];
	        $bit = ['', '', '十', '百', '千', '万', '十', '百', '千', '亿', '十', '百', '千', '万'];
	        $count = strlen($num);
	        $final = '';
	        for($i=$count;$i>0;$i--){
	            $final .= $chinese_num[substr($num, $count-$i, 1)].$bit[$i];
	        }
	        return $final;
	    }
	}

---------------------------------
编译安装phalcon
---------------------------------

安装phalcon前提是需要安装php的pdo，如果使用mysql 需要安装 pdo_mysql

先看下git的版本号
::

	git --version

去存放git目录的位置创建phalcon 的 git，比如我是在home/wwwgit目录下
::

	cd /home/wwwgit/
	git clone git://github.com/phalcon/cphalcon.git

去编译安装目录，注意我的系统是64位，32位的对应去相应的目录
::

	cd cphalcon/build/64bits/
	/usr/local/php/bin/phpize  ./install
	./configure --with-php-config=/usr/local/php/bin/php-config
	make
	make install

如果成功会显示存放目录
::

	Installing shared extensions:     /usr/local/php/lib/php/extensions/no-debug-non-zts-20100525/

ls一下会看到编译的phalcon.so

phalcon.so

在php.ini中加上

extension=phalcon.so



