
========
程序安装
========

--------
环境要求
--------

操作系统
--------

EMQ X采用Erlang/OTP语言平台开发，可跨平台运行在Linux、FreeBSD、Mac OS X、Windows服务器。

产品环境推荐部署在64-bit Linux云主机或服务器。

CPU/内存
--------

EMQ X在测试场景下，1G内存承载80K TCP连接，15K SSL安全连接。

产品部署环境下，建议双机集群，根据并发连接与消息吞吐，规划节点CPU/内存。

----------
程序包命名
----------

EMQ X每个版本会发布Ubuntu、CentOS、FreeBSD、Mac OS X、Windows平台程序包与Docker镜像。

联系EMQ公司获取程序包: http://emqtt.com/about#contacts

程序包命名由平台、版本组成，例如: emqx-enterprise-centos7-v2.1.0.zip

.. _install_rpm:

---------
RPM包安装
---------

CentOS、RedHat操作系统下，推荐RPM包安装。RPM包安装后可通过操作系统，直接管理启停EMQ X服务。

RPM安装
-------

.. code-block:: console

    rpm -ivh --force emqx-centos6.8-v2.1.0-1.el6.x86_64.rpm

.. NOTE:: Erlang/OTP R19依赖lksctp-tools库

.. code-block:: console

    yum install lksctp-tools

配置文件
--------

EMQ X配置文件: /etc/emqx/emqx.conf，插件配置文件: /etc/emqx/plugins/\*.conf。

日志文件
--------

日志文件目录: /var/log/emqx

数据文件
--------

数据文件目录: /var/lib/emqx/

导入License
-----------

导入License文件: cp emqx.lic /etc/emqx/

启动停止
--------

.. code-block:: console

    service emqx start|stop|restart

.. _install_deb:

---------
DEB包安装
---------

Debian、Ubuntu操作系统下，推荐DEB包安装。DEB包安装后可通过操作系统，直接管理启停EMQ X服务。

.. code-block:: console

    sudo dpkg -i emqx-ubuntu16.04_v2.1.0_amd64.deb

.. NOTE:: Erlang/OTP R19依赖lksctp-tools库

.. code-block:: console

    apt-get install lksctp-tools

配置文件
--------

EMQ X配置文件: /etc/emqx/emqx.conf，插件配置文件: /etc/emqx/plugins/\*.conf。

日志文件
--------

日志文件目录: /var/log/emqx

数据文件
--------

数据文件目录: /var/lib/emqx/

导入License
-----------

导入License文件: cp emqx.lic /etc/emqx/


启动停止
--------

.. code-block:: console

    service emqx start|stop|restart

.. _install_on_linux:

---------------
Linux通用包安装
---------------

EMQ X Linux通用程序包:

+---------------------+------------------------------------------+
|  操作系统           |                程序包                    |
+=====================+==========================================+
| CentOS6(64-bit)     | emqx-enterprise-centos6.8-v2.1.0.zip     |
+---------------------+------------------------------------------+
| CentOS7(64-bit)     | emqx-enterprise-centos7-v2.1.0.zip       |
+---------------------+------------------------------------------+
| Ubuntu16.04(64-bit) | emqx-enterprise-ubuntu16.04-v2.1.0.zip   |
+---------------------+------------------------------------------+
| Ubuntu14.04(64-bit) | emqx-enterprise-ubuntu14.04-v2.1.0.zip   |
+---------------------+------------------------------------------+
| Ubuntu12.04(64-bit) | emqx-enterprise-ubuntu12.04-v2.1.0.zip   |
+---------------------+------------------------------------------+
| Debian7(64-bit)     | emqx-enterprise-debian7-v2.1.0.zip       |
+---------------------+------------------------------------------+
| Debian8(64-bit)     | emqx-enterprise-debian8-v2.1.0.zip       |
+---------------------+------------------------------------------+

CentOS平台为例，下载安装过程:

.. code-block:: bash

    unzip emqx-enterprise-centos7-v2.1.0.zip

导入License
-----------

导入License文件 cp emqx.lic etc/


控制台调试模式启动，检查EMQ X是否可正常启动:

.. code-block:: bash

    cd emqx && ./bin/emqx console

如启动正常，控制台输出:

.. code-block:: bash

    Starting emqx on node emqx@127.0.0.1
    Load emqx_mod_presence module successfully.
    Load emqx_mod_subscription module successfully.
    dashboard:http listen on 0.0.0.0:18083 with 2 acceptors.
    mqtt:tcp listen on 127.0.0.1:11883 with 4 acceptors.
    mqtt:tcp listen on 0.0.0.0:1883 with 8 acceptors.
    mqtt:ws listen on 0.0.0.0:8083 with 4 acceptors.
    mqtt:ssl listen on 0.0.0.0:8883 with 4 acceptors.
    mqtt:wss listen on 0.0.0.0:8084 with 4 acceptors.
    emqx 2.1.0 is running now!

CTRL+c关闭控制台。守护进程模式启动:

.. code-block:: bash

    ./bin/emqx start

启动错误日志将输出在log/目录。

EMQ X服务进程状态查询:

.. code-block:: bash

    ./bin/emqx_ctl status

正常运行状态，查询命令返回:

.. code-block:: bash

    $ ./bin/emqx_ctl status
    Node 'emqx@127.0.0.1' is started
    emqx 2.1.0 is running

EMQ X服务器提供了状态监控URL::

    http://localhost:8083/status

停止服务器::

    ./bin/emqx stop

.. _install_on_freebsd:

-----------------
FreeBSD服务器安装
-----------------

联系EMQ公司获取程序包: http://emqtt.com/about#contacts

FreeBSD平台安装过程与Linux相同。

.. _install_on_mac:

----------------
Mac OS X系统安装
----------------

EMQ X Mac平台下安装启动过程与Linux相同。

Mac下开发调试MQTT应用，配置文件'etc/emqx.conf' log段落打开debug日志，控制台可以查看收发MQTT报文详细:

.. code-block:: properties

    ## Console log. Enum: off, file, console, both
    log.console = both

    ## Console log level. Enum: debug, info, notice, warning, error, critical, alert, emergency
    log.console.level = debug

    ## Console log file
    log.console.file = log/console.log

.. _install_docker:

--------------
Docker镜像安装
--------------

EMQ X Docker镜像获取:

解压emqx-enterprise-docker镜像包::

    unzip emqx-enterprise-docker-v2.1.0.zip

加载镜像::

    docker load < emqplus-enterprise-docker-v2.1.0

启动容器::

    docker run -itd --net='host' --name emqx20 emqx-enterprise-docker-v2.1.0

导入License::

    docker cp emqx.lic emqx20:/opt/emqx/etc/


停止容器::

    docker stop emqx20

开启容器::

    docker start emqx20

进入Docker控制台::

    docker exec -it emqx20 /bin/bash

.. _青云:    https://qingcloud.com
.. _AWS:     https://aws.amazon.com
.. _阿里云:  https://www.aliyun.com
.. _UCloud:  https://ucloud.cn
.. _QCloud:  https://www.qcloud.com
.. _HAProxy: https://www.haproxy.org
.. _NGINX:   https://www.nginx.com 

