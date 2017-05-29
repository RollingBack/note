=====================
Sphinx Search
=====================

----------------------
sphinx search 搭建
----------------------

sphinx search 安装
========================

- 获取安装包

.. code-block:: sh

		wget http://sphinxsearch.com/files/sphinx-2.0.10-release.tar.gz

- 解压安装包

.. code-block:: sh

		tar xvfz sphinx-2.0.10-release.tar.gz

- 进入源码目录，配置并编译：

.. code-block:: sh

		cd sphinx-2.0.10-release

		./configure --prefix=/usr/local/sphinx --with-mysql=/usr/local/mysql

		cd api/libsphinxclient/

		./configure --prefix=/usr/local/sphinx

		make && make install

- 安装php扩展

.. code-block:: sh

		wget https://pecl.php.net/get/sphinx-1.3.2.tgz

		tar xvfz sphinx-1.3.2.tgz

		cd sphinx-1.3.2

		/usr/local/php/bin/phpize

		./configure --with-php-config=/usr/local/php/bin/php-config --with-sphinx=/usr/local/sphinx

		make && make install

- 配置PHP

.. code-block:: sh

		vim /usr/local/php/etc/php.ini

加入：

.. code-block:: sh

		extension="sphinx.so"

退出vim，执行下面命令

.. code-block:: sh

		php -m | grep sphinx

如果结果显示sphinx，那么安装成功。

- 重启php-fpm服务。


-------------------------------------
实例
-------------------------------------

因为没有挂分词插件，所以还是算比较简单的。基本原理就是构建一条索引给PHP的Client调用，定期更新（main.sh），同时隔一段时间将构建增量索引并合并到主索引（delta.sh）。官方已经不推荐这种方式，官方推荐使用
实时索引和SphinxSQL，但对于老项目，工作量比较大，这种方式还是比较划算的。这种方式需要一张临时表用于
保存目前建过索引的记录的最大的ID，以便构建增量索引的时候从这个ID开始。至于下面那个很ugly的SQL是因为
Sphinx会把查询的第一个字段当作索引的主键，所以你要给Sphinx提供一个主键，而使用结果集行号作为主键是为
了解决使用ID作为主键后group by之后选取的最上面的记录也就是旧的数据而不是最新数据。因为Sphinx不一定像
MySQL一样稳定，同时作为一个辅助角色，为了系统健壮，还做了Mysql的Like查询，以便于Sphinx挂掉后，系统还能
正常工作。


Sphinx配置文件：
::

	source main_src
	{
		type			= mysql

		sql_host		= sample.com
		sql_user		= root
		sql_pass		= 
		sql_db			= open
		sql_port		= 3306	# optional, default is 3306
		sql_query_pre   = SET NAMES UTF8
		sql_query_pre   = SET SESSION query_cache_type=OFF
		sql_query_pre   = REPLACE INTO sphinx_counter_tmp SELECT 1, MAX(id) FROM tools
		sql_query		= \
			SELECT * FROM \
			    ( \
			        SELECT (@rowNo := @rowNo+1) AS rowno, type_id,if(find_in_set('chinese', language)=1, 1, 0) AS zh, \
			               if(find_in_set('english', language)=1, 1, 0) AS en, \
			               id AS real_id, category, UNIX_TIMESTAMP(insert_time) AS create_time, \
			               UNIX_TIMESTAMP(update_time) AS update_time, title, content, summary_cn, \
			               if(win_link!='',1,0) as win,if(mac_link!='',1,0) as mac, if(win32_link!='' OR win_link!='',1,0) as win32, if(win64_link!='' OR win_link!='',1,0) as win64, hot, new, chosen, develop, \
			               hot_weight, new_weight, chosen_weight, develop_weight, status, score, score_count, price_cn \
			        FROM tools AS a, (select @rowNO :=0) b  WHERE id<=(SELECT max_doc_id FROM sphinx_counter_tmp WHERE counter_id=1) ORDER BY type_id, tools_version DESC \
			    ) AS tools
	    sql_query_info      = SELECT * FROM  tools WHERE id=$id
	    sql_attr_uint		= real_id
		sql_attr_uint		= type_id
		sql_attr_uint		= category
		sql_attr_uint		= win
		sql_attr_uint		= mac
		sql_attr_uint		= win32
		sql_attr_uint		= win64
		sql_attr_uint       = zh
		sql_attr_uint		= en
		sql_attr_uint		= hot
		sql_attr_uint		= new
		sql_attr_uint		= chosen
		sql_attr_uint		= develop
		sql_attr_uint		= hot_weight
		sql_attr_uint		= new_weight
		sql_attr_uint		= chosen_weight
		sql_attr_uint		= develop_weight
		sql_attr_uint		= status
		sql_attr_float      = score
		sql_attr_float      = score_count
		sql_attr_float      = price_cn
		sql_attr_string     = title
		sql_attr_timestamp	= create_time
		sql_attr_timestamp	= update_time
	}

	source delta_src:main_src
	{
		sql_query_pre   = SET NAMES UTF8
		sql_query_pre   = SET SESSION query_cache_type=OFF
		sql_query		= \
			SELECT * FROM \
			    ( \
			        SELECT (@rowNo := @rowNo+1) AS rowno, type_id,if(find_in_set('chinese', language)=1, 1, 0) AS zh, \
			               if(find_in_set('english', language)=1, 1, 0) AS en, \
			               id AS real_id, category, UNIX_TIMESTAMP(insert_time) AS create_time, \
			               UNIX_TIMESTAMP(update_time) AS update_time, title, content, summary_cn, \
			               if(win_link!='',1,0) as win,if(mac_link!='',1,0) as mac, if(win32_link!='' OR win_link!='',1,0) as win32, if(win64_link!='' OR win_link!='',1,0) as win64, hot, new, chosen, develop, \
			               hot_weight, new_weight, chosen_weight, develop_weight, status, score, score_count, price_cn \
			        FROM tools AS a, (select @rowNO :=0) b WHERE id>(SELECT max_doc_id FROM sphinx_counter_tmp WHERE counter_id=1) ORDER BY type_id, tools_version DESC \
			    ) AS tools

		sql_attr_uint		= real_id
		sql_attr_uint		= type_id
		sql_attr_uint		= category
		sql_attr_uint		= win
		sql_attr_uint		= mac
		sql_attr_uint		= win32
		sql_attr_uint		= win64
		sql_attr_uint       = zh
		sql_attr_uint		= en
		sql_attr_uint		= hot
		sql_attr_uint		= new
		sql_attr_uint		= chosen
		sql_attr_uint		= develop
		sql_attr_uint		= hot_weight
		sql_attr_uint		= new_weight
		sql_attr_uint		= chosen_weight
		sql_attr_uint		= develop_weight
		sql_attr_uint		= status
		sql_attr_float      = score
		sql_attr_float      = score_count
		sql_attr_float      = price_cn
		sql_attr_string     = title
		sql_attr_timestamp	= create_time
		sql_attr_timestamp	= update_time
	}

	index main
	{
		source			= main_src
		path			= /var/lib/sphinx/data/main
		docinfo			= extern
		charset_type		= utf-8
		charset_table           = U+FF10..U+FF19->0..9, 0..9, U+FF41..U+FF5A->a..z, U+FF21..U+FF3A->a..z,\
	A..Z->a..z, a..z, U+0149, U+017F, U+0138, U+00DF, U+00FF, U+00C0..U+00D6->U+00E0..U+00F6,\
	U+00E0..U+00F6, U+00D8..U+00DE->U+00F8..U+00FE, U+00F8..U+00FE, U+0100->U+0101, U+0101,\
	U+0102->U+0103, U+0103, U+0104->U+0105, U+0105, U+0106->U+0107, U+0107, U+0108->U+0109,\
	U+0109, U+010A->U+010B, U+010B, U+010C->U+010D, U+010D, U+010E->U+010F, U+010F,\
	U+0110->U+0111, U+0111, U+0112->U+0113, U+0113, U+0114->U+0115, U+0115, \
	U+0116->U+0117,U+0117, U+0118->U+0119, U+0119, U+011A->U+011B, U+011B, U+011C->U+011D,\
	U+011D,U+011E->U+011F, U+011F, U+0130->U+0131, U+0131, U+0132->U+0133, U+0133, \
	U+0134->U+0135,U+0135, U+0136->U+0137, U+0137, U+0139->U+013A, U+013A, U+013B->U+013C, \
	U+013C,U+013D->U+013E, U+013E, U+013F->U+0140, U+0140, U+0141->U+0142, U+0142, \
	U+0143->U+0144,U+0144, U+0145->U+0146, U+0146, U+0147->U+0148, U+0148, U+014A->U+014B, \
	U+014B,U+014C->U+014D, U+014D, U+014E->U+014F, U+014F, U+0150->U+0151, U+0151, \
	U+0152->U+0153,U+0153, U+0154->U+0155, U+0155, U+0156->U+0157, U+0157, U+0158->U+0159,\
	U+0159,U+015A->U+015B, U+015B, U+015C->U+015D, U+015D, U+015E->U+015F, U+015F, \
	U+0160->U+0161,U+0161, U+0162->U+0163, U+0163, U+0164->U+0165, U+0165, U+0166->U+0167, \
	U+0167,U+0168->U+0169, U+0169, U+016A->U+016B, U+016B, U+016C->U+016D, U+016D, \
	U+016E->U+016F,U+016F, U+0170->U+0171, U+0171, U+0172->U+0173, U+0173, U+0174->U+0175,\
	U+0175,U+0176->U+0177, U+0177, U+0178->U+00FF, U+00FF, U+0179->U+017A, U+017A, \
	U+017B->U+017C,U+017C, U+017D->U+017E, U+017E, U+0410..U+042F->U+0430..U+044F, \
	U+0430..U+044F,U+05D0..U+05EA, U+0531..U+0556->U+0561..U+0586, U+0561..U+0587, \
	U+0621..U+063A, U+01B9,U+01BF, U+0640..U+064A, U+0660..U+0669, U+066E, U+066F, \
	U+0671..U+06D3, U+06F0..U+06FF,U+0904..U+0939, U+0958..U+095F, U+0960..U+0963, \
	U+0966..U+096F, U+097B..U+097F,U+0985..U+09B9, U+09CE, U+09DC..U+09E3, U+09E6..U+09EF, \
	U+0A05..U+0A39, U+0A59..U+0A5E,U+0A66..U+0A6F, U+0A85..U+0AB9, U+0AE0..U+0AE3, \
	U+0AE6..U+0AEF, U+0B05..U+0B39,U+0B5C..U+0B61, U+0B66..U+0B6F, U+0B71, U+0B85..U+0BB9, \
	U+0BE6..U+0BF2, U+0C05..U+0C39,U+0C66..U+0C6F, U+0C85..U+0CB9, U+0CDE..U+0CE3, \
	U+0CE6..U+0CEF, U+0D05..U+0D39, U+0D60,U+0D61, U+0D66..U+0D6F, U+0D85..U+0DC6, \
	U+1900..U+1938, U+1946..U+194F, U+A800..U+A805,U+A807..U+A822, U+0386->U+03B1, \
	U+03AC->U+03B1, U+0388->U+03B5, U+03AD->U+03B5,U+0389->U+03B7, U+03AE->U+03B7, \
	U+038A->U+03B9, U+0390->U+03B9, U+03AA->U+03B9,U+03AF->U+03B9, U+03CA->U+03B9, \
	U+038C->U+03BF, U+03CC->U+03BF, U+038E->U+03C5,U+03AB->U+03C5, U+03B0->U+03C5, \
	U+03CB->U+03C5, U+03CD->U+03C5, U+038F->U+03C9,U+03CE->U+03C9, U+03C2->U+03C3, \
	U+0391..U+03A1->U+03B1..U+03C1,U+03A3..U+03A9->U+03C3..U+03C9, U+03B1..U+03C1, \
	U+03C3..U+03C9, U+0E01..U+0E2E,U+0E30..U+0E3A, U+0E40..U+0E45, U+0E47, U+0E50..U+0E59, \
	U+A000..U+A48F, U+4E00..U+9FBF,U+3400..U+4DBF, U+20000..U+2A6DF, U+F900..U+FAFF, \
	U+2F800..U+2FA1F, U+2E80..U+2EFF,U+2F00..U+2FDF, U+3100..U+312F, U+31A0..U+31BF, \
	U+3040..U+309F, U+30A0..U+30FF,U+31F0..U+31FF, U+AC00..U+D7AF, U+1100..U+11FF, \
	U+3130..U+318F, U+A000..U+A48F,U+A490..U+A4CF
		min_prefix_len = 0
		min_infix_len = 1
		ngram_len = 1
	}

	index delta:main
	{
		source			= delta_src
		path			= /var/lib/sphinx/data/delta
		docinfo			= extern
		charset_type		= utf-8
		charset_table           = U+FF10..U+FF19->0..9, 0..9, U+FF41..U+FF5A->a..z, U+FF21..U+FF3A->a..z,\
	A..Z->a..z, a..z, U+0149, U+017F, U+0138, U+00DF, U+00FF, U+00C0..U+00D6->U+00E0..U+00F6,\
	U+00E0..U+00F6, U+00D8..U+00DE->U+00F8..U+00FE, U+00F8..U+00FE, U+0100->U+0101, U+0101,\
	U+0102->U+0103, U+0103, U+0104->U+0105, U+0105, U+0106->U+0107, U+0107, U+0108->U+0109,\
	U+0109, U+010A->U+010B, U+010B, U+010C->U+010D, U+010D, U+010E->U+010F, U+010F,\
	U+0110->U+0111, U+0111, U+0112->U+0113, U+0113, U+0114->U+0115, U+0115, \
	U+0116->U+0117,U+0117, U+0118->U+0119, U+0119, U+011A->U+011B, U+011B, U+011C->U+011D,\
	U+011D,U+011E->U+011F, U+011F, U+0130->U+0131, U+0131, U+0132->U+0133, U+0133, \
	U+0134->U+0135,U+0135, U+0136->U+0137, U+0137, U+0139->U+013A, U+013A, U+013B->U+013C, \
	U+013C,U+013D->U+013E, U+013E, U+013F->U+0140, U+0140, U+0141->U+0142, U+0142, \
	U+0143->U+0144,U+0144, U+0145->U+0146, U+0146, U+0147->U+0148, U+0148, U+014A->U+014B, \
	U+014B,U+014C->U+014D, U+014D, U+014E->U+014F, U+014F, U+0150->U+0151, U+0151, \
	U+0152->U+0153,U+0153, U+0154->U+0155, U+0155, U+0156->U+0157, U+0157, U+0158->U+0159,\
	U+0159,U+015A->U+015B, U+015B, U+015C->U+015D, U+015D, U+015E->U+015F, U+015F, \
	U+0160->U+0161,U+0161, U+0162->U+0163, U+0163, U+0164->U+0165, U+0165, U+0166->U+0167, \
	U+0167,U+0168->U+0169, U+0169, U+016A->U+016B, U+016B, U+016C->U+016D, U+016D, \
	U+016E->U+016F,U+016F, U+0170->U+0171, U+0171, U+0172->U+0173, U+0173, U+0174->U+0175,\
	U+0175,U+0176->U+0177, U+0177, U+0178->U+00FF, U+00FF, U+0179->U+017A, U+017A, \
	U+017B->U+017C,U+017C, U+017D->U+017E, U+017E, U+0410..U+042F->U+0430..U+044F, \
	U+0430..U+044F,U+05D0..U+05EA, U+0531..U+0556->U+0561..U+0586, U+0561..U+0587, \
	U+0621..U+063A, U+01B9,U+01BF, U+0640..U+064A, U+0660..U+0669, U+066E, U+066F, \
	U+0671..U+06D3, U+06F0..U+06FF,U+0904..U+0939, U+0958..U+095F, U+0960..U+0963, \
	U+0966..U+096F, U+097B..U+097F,U+0985..U+09B9, U+09CE, U+09DC..U+09E3, U+09E6..U+09EF, \
	U+0A05..U+0A39, U+0A59..U+0A5E,U+0A66..U+0A6F, U+0A85..U+0AB9, U+0AE0..U+0AE3, \
	U+0AE6..U+0AEF, U+0B05..U+0B39,U+0B5C..U+0B61, U+0B66..U+0B6F, U+0B71, U+0B85..U+0BB9, \
	U+0BE6..U+0BF2, U+0C05..U+0C39,U+0C66..U+0C6F, U+0C85..U+0CB9, U+0CDE..U+0CE3, \
	U+0CE6..U+0CEF, U+0D05..U+0D39, U+0D60,U+0D61, U+0D66..U+0D6F, U+0D85..U+0DC6, \
	U+1900..U+1938, U+1946..U+194F, U+A800..U+A805,U+A807..U+A822, U+0386->U+03B1, \
	U+03AC->U+03B1, U+0388->U+03B5, U+03AD->U+03B5,U+0389->U+03B7, U+03AE->U+03B7, \
	U+038A->U+03B9, U+0390->U+03B9, U+03AA->U+03B9,U+03AF->U+03B9, U+03CA->U+03B9, \
	U+038C->U+03BF, U+03CC->U+03BF, U+038E->U+03C5,U+03AB->U+03C5, U+03B0->U+03C5, \
	U+03CB->U+03C5, U+03CD->U+03C5, U+038F->U+03C9,U+03CE->U+03C9, U+03C2->U+03C3, \
	U+0391..U+03A1->U+03B1..U+03C1,U+03A3..U+03A9->U+03C3..U+03C9, U+03B1..U+03C1, \
	U+03C3..U+03C9, U+0E01..U+0E2E,U+0E30..U+0E3A, U+0E40..U+0E45, U+0E47, U+0E50..U+0E59, \
	U+A000..U+A48F, U+4E00..U+9FBF,U+3400..U+4DBF, U+20000..U+2A6DF, U+F900..U+FAFF, \
	U+2F800..U+2FA1F, U+2E80..U+2EFF,U+2F00..U+2FDF, U+3100..U+312F, U+31A0..U+31BF, \
	U+3040..U+309F, U+30A0..U+30FF,U+31F0..U+31FF, U+AC00..U+D7AF, U+1100..U+11FF, \
	U+3130..U+318F, U+A000..U+A48F,U+A490..U+A4CF
		min_prefix_len = 0
		min_infix_len = 1
		ngram_len = 1
	}

	indexer
	{
		mem_limit		= 256M
	}


	searchd
	{
		listen			= 9312
		listen			= 9306:mysql41
		log				= /var/log/sphinx/searchd.log
		query_log		= /var/log/sphinx/query.log
		read_timeout	= 1
		max_children	= 30
		pid_file		= /var/run/sphinx/searchd.pid
		max_matches		= 1000
		seamless_rotate	= 1
		preopen_indexes	= 1
		unlink_old		= 1
		workers			= threads # for RT to work
		binlog_path		= /var/lib/sphinx/data
		compat_sphinxql_magics=0
	}


main.sh

.. code-block:: sh

	#!/bin/bash
	/usr/local/sphinx/bin/indexer main -c /usr/local/sphinx/etc/sphinx.conf --rotate >> /var/log/sphinx/main.log

delta.sh

.. code-block:: sh

	#!/bin/bash
	/usr/local/sphinx/bin/indexer delta -c /usr/local/sphinx/etc/sphinx.conf --rotate >> /var/log/sphinx/delta.log
	/usr/local/sphinx/bin/indexer --merge main delta -c /usr/local/sphinx/etc/sphinx.conf --rotate >> /var/log/sphinx/delta.log

searcher.php

.. code-block:: php

	<?php

	/**
	 * Class Search_model
	 */
	class Searcher extends CI_Model{


	    /**
	     * @var SphinxClient
	     */
	    private $sphinx;

	    /**
	     *
	     */
	    public function __construct(){
	        parent::__construct();
	        //if Sphinx php extension is installed use sphinx for search
	        if(class_exists('SphinxClient')){
	            $this->sphinx = new SphinxClient();
	            //connect to sphinx server
	            $this->sphinx->setServer(
	            	$this->config->item('sphinx')['host'], 
	            	$this->config->item('sphinx')['port']
	            );
	            $this->sphinx->setMaxQueryTime(
	            	$this->config->item('sphinx')['max_query_time']
	            );
	            //if the sphinx server is gone after 3s the connect shut down
	            $this->sphinx->setConnectTimeout(3);
	            $this->sphinx->setMatchMode(SPH_MATCH_EXTENDED2);
	        }
	    }

	    
	    /**
	     * search in sphinx index, then use find_in_set get data in mysql, different from like way, 
	     *	it has a 1000 default limit
	     *
	     * @param $language
	     * @param $platform
	     * @param $keyword
	     * @param $category
	     * @param $start
	     * @param $limit
	     *
	     * @return array
	     */
	    public function search_in_sphinx($language, $platform,$keyword, $category, $start, $limit){
	        if($keyword==''){
	            return ;
	        }
	        $platform = strtolower($platform);
	        //is available or not
	        $this->sphinx->setFilter('status', array(1));
	        //win or mac?
	        if(in_array($platform, array('win', 'mac', 'win32', 'win64'))){
	            $this->sphinx->setFilter($platform, array(1));
	        }
	        //zh or en?
	        $this->sphinx->setFilter($language, array(1));
	        if($category!=0){
	            $this->sphinx->setFilter(
		            'category',
		            array_merge(array($category), 
		            (array)$this->get_cat_sons($category, FALSE))
	            );
	        }
	        //set group by type_id
	        $this->sphinx->setGroupBy(
	        	'type_id', 
	        	SPH_GROUPBY_ATTR, 
	        	'hot DESC, 
	        	new DESC, 
	        	update_time DESC, 
	        	head'
	        );
	        $this->sphinx->setRankingMode(SPH_RANK_NONE);
	        //set limit , max is 1000, if is 0 then you will get nothing
	        $this->sphinx->setLimits($start, $limit, 1000);
	        $this->sphinx->setArrayResult(True);
	        //fetch in main index
	        $result = $this->sphinx->query($keyword, 'main');
	        //get real_id from result, the id in sphinx is the row number of the mysql result
	        $ids = array();
	        foreach($result['matches'] as $val){
	            $ids[] = $val['attrs']['real_id'];
	        }
	        if(count($ids)<1){
	            return ;
	        }
	        $data = array();
	        $data['total'] = $result['total'];

	        //get the final result from mysql server
	        $sql = ' SELECT * FROM table_name
	                 WHERE `id` in ('.join(',', $ids).')  
	                 ORDER BY field(id, '.join(',', $ids).') ';
	        $data['data'] = $this->db->query($sql)->result_array();
	        return $data;
	    }    

	}