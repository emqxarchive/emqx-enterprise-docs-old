
.. _install:

========
安装部署
========

*EMQ* 企业版R2服务器可跨平台运行在Linux、FreeBSD、Mac OS X、Windows服务器或树莓派上。

.. NOTE:: 产品部署建议Linux、FreeBSD服务器，不推荐Windows服务器。

-----------------
EMQ企业版R2程序包
-----------------

*EMQ* 企业版服务器每个版本会发布Ubuntu、CentOS、FreeBSD、Mac OS X、Windows平台程序包与Docker镜像。

联系EMQ公司获取程序包: http://emqtt.com/about#contacts

安装包命名由平台、版本组成，例如: emqx-enterprise-centos7-r2.1.0.zip

.. _install_on_linux:

---------------
Linux服务器安装
---------------

CentOS平台为例，获取安装包: emqx-enterprise-centos7-v2.1.0.zip

.. code-block:: bash

    unzip emqx-enterprise-centos7-r2.1.0.zip

控制台调试模式启动，检查 *EMQ X* 是否可正常启动:

.. code-block:: bash

    cd emqx && ./bin/emqx console

*EMQ X* 服务器如启动正常，控制台输出:

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

*EMQ X* 服务器进程状态查询:

.. code-block:: bash

    ./bin/emqx_ctl status

正常运行状态，查询命令返回:

.. code-block:: bash

    $ ./bin/emqx_ctl status
    Node 'emqx@127.0.0.1' is started
    emqx 2.1.0 is running

*EMQ X* 服务器提供了状态监控URL::

    http://localhost:8083/status

停止服务器::

    ./bin/emqx stop

.. _install_rpm_on_linux:

---------
RPM包安装
---------

CentOS、RedHat操作系统下，建议直接安装RPM包。RPM包安装后可通过操作系统，直接管理启停EMQ服务。

RPM安装
-------

.. code-block::

    rpm -ivh --force emqx-centos6.8-v2.1.0-1.el6.x86_64.rpm

配置文件
--------

*EMQ X* 配置文件: /etc/emqx/emqx.conf，插件配置文件/etc/emqx/plugins/\*.conf。

日志文件
--------

日志文件目录: /var/log/emqx

数据文件
--------

数据文件目录：/var/lib/emqx/

启动停止
--------

.. code-block::

    service emqx start|stop|restart

---------
DEB包安装
---------

Debian、Ubuntu操作系统下，建议直接安装DEB包。DEB包安装后可通过操作系统，直接管理启停EMQ服务。

.. code-block::

    sudo dpkg -i emqx-ubuntu16.04_v2.1.0_amd64.deb

配置文件
--------

*EMQ X* 配置文件: /etc/emqx/emqx.conf，插件配置文件/etc/emqx/plugins/\*.conf。

日志文件
--------

日志文件目录: /var/log/emqx

数据文件
--------

数据文件目录：/var/lib/emqx/

启动停止
--------

.. code-block::

    service emqx start|stop|restart

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

*EMQ* 企业版在Mac平台下安装启动过程与Linux相同。

Mac下开发调试MQTT应用，配置文件'etc/emqx.conf' log段落打开debug日志，控制台可以查看收发MQTT报文详细:

.. code-block::

    ## Console log. Enum: off, file, console, both
    log.console = both

    ## Console log level. Enum: debug, info, notice, warning, error, critical, alert, emergency
    log.console.level = debug

    ## Console log file
    log.console.file = log/console.log

.. _install_on_windows:

-----------------
Windows服务器安装
-----------------

Windows平台程序包获取解压后，打开Windows命令行窗口，cd到程序目录。

控制台模式启动::

    bin\emqx console

如启动成功，会弹出控制台窗口。

关闭控制台窗口，停止emqttd进程，准备注册Windows服务。

.. WARNING:: EMQX不支持服务注册

*EMQ X* 注册为Windows服务::

    bin\emqx install

*EMQ X* 服务启动::

    bin\emqx start

*EMQ X* 服务停止::

    bin\emqx stop

*EMQ X* 服务卸载::

    bin\emqx uninstall

.. _install_docker:

--------------
Docker镜像安装
--------------

EMQ企业版R2 Docker镜像获取:

解压emqx-enterprise-docker镜像包::

    unzip emqx-enterprise-docker-v2.1.0.zip

加载镜像::

    docker load < emqplus-enterprise-docker-v2.1.0

启动容器::

    docker run -itd --net='host' --name emqx20 emqx-enterprise-docker-v2.1.0

停止容器::

    docker stop emqx20

开启容器::

    docker start emqx20

进入Docker控制台::

    docker exec -it emqx20 /bin/bash

.. _tcp_ports:

---------------
TCP服务端口占用
---------------

EMQ企业版服务器默认启用的外部MQTT服务端口包括:

+-----------+-----------------------------------+
| 1883      | MQTT协议端口                      |
+-----------+-----------------------------------+
| 8883      | MQTT/SSL端口                      |
+-----------+-----------------------------------+
| 8083      | MQTT/WebSocket端口                |
+-----------+-----------------------------------+
| 8084      | MQTT/WebSocket(SSL)端口           |
+-----------+-----------------------------------+
| 18083     | Dashboard管理控制台端口           |
+-----------+-----------------------------------+

上述占用端口可通过etc/emqx.conf配置文件的'Listeners'段落设置:

.. code-block:: properties

    ## External TCP Listener: 1883, 127.0.0.1:1883, ::1:1883
    listener.tcp.external = 0.0.0.0:1883

    ## SSL Listener: 8883, 127.0.0.1:8883, ::1:8883
    listener.ssl.external = 8883
    
    ## HTTP and WebSocket Listener
    listener.http.external = 8083

    ## External HTTPS and WSS Listener
    listener.https.external = 8084

通过注释或删除相关段落，可禁用相关TCP服务启动。

18083端口是Web管理控制占用，该端口由'emqx-dashboard'插件启用。

控制台URL: http:://localhost:18083/ ，默认登录用户名: admin, 密码: public。

------------
集群端口占用
------------

EMQ企业版服务器集群，使用的TCP端口包括: 

+-----------+-----------------------------------+
| 4369      | 集群节点发现端口                  |
+-----------+-----------------------------------+
| 5369      | 集群节点数据通道                  |
+-----------+-----------------------------------+
| 6369      | 集群节点控制通道                  |
+-----------+-----------------------------------+

.. _quick_setup:

--------
快速设置
--------

*EMQ X* 服务器主要配置文件:

+-----------------------+-----------------------------------+
| etc/emqx.conf         | EMQ企业版服务器参数设置           |
+-----------------------+-----------------------------------+
| etc/plugins/\*.conf   | EMQ企业版插件配置文件             |
+-----------------------+-----------------------------------+

etc/emqx.conf 中两个重要的虚拟机启动参数:

+-----------------------+------------------------------------------------------------------+
| node.process_limit    | Erlang虚拟机允许的最大进程数，emqttd一个连接会消耗2个Erlang进程  |
+-----------------------+------------------------------------------------------------------+
| node.max_ports        | Erlang虚拟机允许的最大Port数量，emqttd一个连接消耗1个Port        |
+-----------------------+------------------------------------------------------------------+

.. NOTE:: Erlang的Port非TCP端口，可以理解为文件句柄。

node.process_limit = 参数值 > 最大允许连接数 * 2

node.max_ports = 参数值 > 最大允许连接数

.. WARNING:: 实际连接数量超过Erlang虚拟机参数设置，会引起EMQ消息服务器宕机!

etc/emqx.conf配置文件的'Listeners`段落设置最大允许连接数:

.. code-block:: properties

    listener.tcp.external = 0.0.0.0:1883

    listener.tcp.external.acceptors = 8

    listener.tcp.external.max_clients = 1024

EMQ企业版服务器详细设置，请参见文档: :ref:`config`

