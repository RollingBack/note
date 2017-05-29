=====================================
版本控制工具
=====================================

------------------------------------
SVN服务器搭建
------------------------------------

**1.** 安装subversion,苹果系统自带，当然也可以用brew安装最新的。

.. code-block:: sh

	brew install subversion

**2.** 新建个目录用于存放版本库

.. code-block:: sh
	
	cd ~
	mkdir svn

**3.** 新建个版本仓库

.. code-block:: sh
	
	svnadmin create ~/svn/myproject

**4.** 添加用户

.. code-block:: sh
	
	vim ~/svn/myproject/conf/passwd

加载下面添加：
::

	user = password
	test = 123456

**5.** 修改用户访问策略
::
	
	vim ~/svn/myproject/conf/authz

添加：
::

	[groups]
	admin = user,test
	[myproject:/]
	@admin = rw

**6.** 修改svnserve.conf，让配置生效。
::

	[general]
	anon-access = none
	auth-access = write
	password-db = passwd
	authz-db    = authz

**7.** 启动服务器
::

	svnserve -d -r ~/svn

------------------------------------
SVN的坑
------------------------------------

**1.** 分支合并到主干上时代码冲突多。

你需要先反向合并。还有就是合并前自己要检查代码，或者干脆用好的对比软件手动合并。

-------------------------------------
GIT
-------------------------------------

--------------------------------------
CVS
--------------------------------------

