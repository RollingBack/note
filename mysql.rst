=========================================
MySQL
=========================================

----------------------------------------
mysql的100个坑
----------------------------------------

**1.** 乱码。
用windows的同志一定要记得set names utf8。

**2.** 备份。
在对数据库进行操作时记得备份，很简单的道理，多少人挨坑。

**3.** 外部连接。

从外部连接记得配置防火墙，绑定IP地址。

实例：

ubuntu server下安装了MySQL 5.5数据库，然后在windows下通过Navicat for MySQL连接时，出现 Can't connect to mysql server on xxx.xxx.xxx.xxx(10038) 的问题。

解决方案如下：

    - 授权

    .. code-block:: sh

        mysql>grant all privileges on *.*  to  'root'@'%'  identified by 'youpassword'  with grant option;
        mysql>flush privileges;

    - 修改/etc/mysql/my.conf

    找到bind-address = 127.0.0.1这一行

    改为bind-address = 0.0.0.0即可

**4.** SQL_CALC_FOUND_ROWS
SELECT FOUND_ROWS();可以取到结果集条数。SQL语句比较复杂的情况下索引无效，性能可能不如count(*)。

**5.** 没建索引，或者变更代码时没考虑索引的问题。
记得建索引。不要让你的表有太多的字段，因为太多的字段十有八九意味着你的SQL会变得很复杂，甚至没法建索引。

**6.** NULL。
ifnull(`fieldname`,0) 如果fieldname为空值null,那么返回0
如果允许字段值为null,where语句中只能使用 is null, is not null进行空值判定，!=""和==""不起作用

**7.** 表集合。
两种语法：
第一种，union会试图使集合的表中的数据唯一

.. code-block:: sql

    select `fieldname` from table1 where ...
    union
    select `fieldname` from table2 where ...

第二种，union all，直接集合不做任何处理

.. code-block:: sql

    select `fieldname` from table1 where ...
    union all
    select `fieldname` from table2 where ...

集合的字段类型应该相同

**8.** 还在用PHP生成CSV文件，或者导入数据？你OUT了！下面的方法快如闪电，结合CONCAT，SUBSTR等MySQL原生函数，几百万数据的XML文件绝对分分钟搞定，我用PHP导出时耗时长，最后因为占用太高挂了。当然你可以优化代码，做拼接。

.. code-block:: sql

    LOAD DATA INFILE "c:/sss.csv" INTO TABLE `table_name` CHARACTER SET utf8 LINES
    TERMINATED BY '\r' FIELDS TERMINATED BY ',';
    SELECT * INTO OUTFILE 'file_name' FROM tbl_name where 1;

**9.** 数据库位置迁移（Ubuntu）。不要以为很简单，这里面有个坑，防火墙。

.. code-block:: sh
    :linenos:

    /etc/init.d/mysql stop                #停止mysql服务
    vim /etc/mysql/my.cnf                 #编辑mysql配置文件
        datadir = /data/mysql             #修改datadir选项位置
    mv -R /var/lib/mysql/ /data/          #把数据库文件放到新的位置
    chown -R mysql:mysql /data            #确保新位置的用户和用户组及权限正确
    vim /etc/apparmor.d/usr.sbin.mysqld   #编辑防火墙，注释以前位置，加入新位置
        #/var/lib/mysql/ r,
        #/var/lib/mysql/** rwk,
        /data/mysql/ r,
        /data/mysql/** rwk,
    /etc/init.d/apparmor reload           #使防火墙配置生效
    /etc/init.d/mysql start               #启动mysql

**10.** 使用了无法缓存的语法。
含有NOW(),CURDATE(),RAND()的SQL无法缓存，尽量避免使用。

**11.** 表连接效率低下。
首先explain查看索引使用的情况。最糟糕的是type是ALL，如果通过加索引使用联合索引不能解决，尝试使用PHP处理部分逻辑。


------------------------------------
MySQL主从库设置
------------------------------------

**1.**  主从服务器分别作以下操作：

- 保证版本一致

- 初始化表，并在后台启动mysql

- 修改root的密码

**2.**  修改主服务器master:
::

    #vi /etc/my.cnf
    [mysqld]
    log-bin=mysql-bin   //[必须]启用二进制日志
    server-id=222       //[必须]服务器唯一ID，默认是1，一般取IP最后一段

**3.**  修改从服务器slave:
::

    #vi /etc/my.cnf
    [mysqld]
    relay_log=relay-log   //[必须]启用relay日志
    server-id=226       //[必须]服务器唯一ID，默认是1，一般取IP最后一段

**4.**  重启两台服务器的mysql
::

   /etc/init.d/mysql restart

**5.**  在主服务器上建立帐户并授权slave:
::

   #/usr/local/mysql/bin/mysql -uroot -pmttang
   mysql>GRANT REPLICATION SLAVE ON *.* to 'mysync'@'%' identified by 'password'; //一般不用root帐号，“%”表示所有客户端都可能连，只要帐号，密码正确，此处可用具体客户端IP代替，如192.168.145.226，加强安全。

**6.**  登录主服务器的mysql，查询master的状态
::

   mysql>show master status;
   +------------------+----------+--------------+------------------+
   | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
   +------------------+----------+--------------+------------------+
   | mysql-bin.000004 |      308 |              |                  |
   +------------------+----------+--------------+------------------+
   1 row in set (0.00 sec)
   注：执行完此步骤后不要再操作主服务器MYSQL，防止主服务器状态值变化, 必要时请锁库

**7.**  配置从服务器Slave：
::

   mysql>change master to aster_host='192.168.145.222',master_user='mysync',master_password='password',
         master_log_file='mysql-bin.000004',master_log_pos=308;   //注意不要断开，“308”无单引号,为step6中主服务器中position里的数值。

   Mysql>start slave;    //启动从服务器复制功能

**8.**  检查从服务器复制功能状态：
::

   mysql> show slave status\G

   *************************** 1. row ***************************

                Slave_IO_State: Waiting for master to send event

                   Master_Host: 192.168.2.222  //主服务器地址

                   Master_User: myrync         //授权帐户名，尽量避免使用root

                   Master_Port: 3306           //数据库端口，部分版本没有此行

                 Connect_Retry: 60

               Master_Log_File: mysql-bin.000004

           Read_Master_Log_Pos: 600        //#同步读取二进制日志的位置，大于等于>=Exec_Master_Log_Pos

                Relay_Log_File: ddte-relay-bin.000003

                 Relay_Log_Pos: 251

         Relay_Master_Log_File: mysql-bin.000004

              Slave_IO_Running: Yes       //此状态必须YES

             Slave_SQL_Running: Yes       //此状态必须YES
                    ......

注：Slave_IO及Slave_SQL进程必须正常运行，即YES状态，否则都是错误的状态(如：其中一个NO均属错误)。

以上操作过程，主从服务器配置完成。

**9.**  主从服务器测试：

主服务器Mysql，建立数据库，并在这个库中建表插入一条数据：
::

  mysql> create database hi_db;
  Query OK, 1 row affected (0.00 sec)

  mysql> use hi_db;
  Database changed

  mysql>  create table hi_tb(id int(3),name char(10));
  Query OK, 0 rows affected (0.00 sec)

  mysql> insert into hi_tb values(001,'bobu');
  Query OK, 1 row affected (0.00 sec)

  mysql> show databases;
   +--------------------+
   | Database           |
   +--------------------+
   | information_schema |
   | hi_db              |
   | mysql              |
   | test               |
   +--------------------+
   4 rows in set (0.00 sec)

从服务器Mysql查询：
::

   mysql> show databases;

   +--------------------+
   | Database           |
   +--------------------+
   | information_schema |
   | hi_db              |          //I'M here，大家看到了吧
   | mysql              |
   | test               |
   +--------------------+
   4 rows in set (0.00 sec)

   mysql> use hi_db
   Database changed
   mysql> select * from hi_tb;    //可以看到在主服务器上新增的具体数据
   +------+------+
   | id   | name |
   +------+------+
   |    1 | bobu |
   +------+------+
   1 row in set (0.00 sec)

**10.** 完成：
    编写一shell脚本，用nagios监控slave的两个“yes”，如发现只有一个或零个“yes”，就表明主从有问题了，发短信警报吧。

------------------------------------
阿里云MySQL主从库设置
------------------------------------

**1.** 主库设置：

- 从阿里云售后工程师处得知，具有读写权限的账号具有复制权限，建立一个具有读写权限的账号；

- 把从库的IP加入主库白名单

**2.** 从库设置：

- 选用官方版本编译安装，图简便可以用lnmp.org的包，安装的mysql版本必须高于等于主库版本；

- 从阿里云的控制台的备份恢复处下载最新的备份和最新的binlog;

- 按照 https://help.aliyun.com/knowledge_detail/41817.html 的操作流程，把数据备份恢复到从库。过程中，不要完全按照指示进行恢复，把备份中的my.cnf合并到现在的/etc/my.cnf下面的几个选项要注释掉，不然mysql无法启动：

::

        #innodb_checksum_algorithm=innodb
        #innodb_log_checksum_algorithm=innodb
        #innodb_encrypt_algorithm=aes_128_ecb

还需要设置一个不同于主库的server_id。

- 执行完恢复工作，执行mysql_upgrade，把数据升级成从库版本。

- 恢复binlog，mysqlbinlog mysql-bin.**** | mysql -u root -p

- 修改启动选项增加如下选项：--gtid_mode=ON --log-bin --log-slave-update --enforce-gtid-consistency

- 启动mysql，进入mysql终端，把master指向主库：

::

    change master to
        master_host='#阿里云主库的hostname',
        master_user='#主库设置第一步中的账号',
        master_password='#主库设置第一步中的账号的密码',
        master_port=3306,
        master_log_file='#从库设置中下载并导入的binlog的下一个binlog',
        master_log_pos=4#未经证实，但是似乎阿里云的binlog位置都是从4开始的，可以用mysqlbinlog查看一下主库的前几行;

- 终端中输入start slave;开始同步。如果有报错，需要手动更新部分数据表。然后stop slave; start slave;



