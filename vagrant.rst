=========================================
vagrant
=========================================

---------------------------------------
安装
---------------------------------------

**1.** 安装 `virtualbox`_.


**2.** 安装 `vagrant`_.


**3.** windows需要安装一个支持ssh的shell, 推荐安装 `Cygwin`_.



**4.** 从 `vagrant boxes`_ 挑选合适的box.

**5.** 在shell中执行:

.. code-block:: sh

    vagrant init ubuntu/trusty64;
    vagrant up --provider virtualbox

如果网速很慢或者下载失败, 可以手动下载并添加.

.. code-block:: sh

    vagrant box add mydev ubuntu.box
    vagrant init mydev
    vagrant up

**6.** 通过ssh接入环境:

.. code-block:: sh

    vagrant ssh


.. _virtualbox: https://www.virtualbox.org/
.. _vagrant: https://www.vagrantup.com/downloads.html
.. _Cygwin: https://www.cygwin.com/
.. _vagrant boxes: https://atlas.hashicorp.com/boxes/search


**7.** 系统更新后无法共享文件夹的解决方法

.. code-block:: sh

    sudo /etc/init.d/vboxadd setup