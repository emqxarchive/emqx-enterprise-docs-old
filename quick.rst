

.. _quick:

========
快速启动
========

假设部署两台EMQ X Linux节点集群，在云厂商VPC或私有网络内:

+---------------------+---------------------+
| 节点名              |    IP地址           |
+---------------------+---------------------+
| emqx1@192.168.0.10  | 192.168.0.10        |
+---------------------+---------------------+
| emqx2@192.168.0.20  | 192.168.0.20        |
+---------------------+---------------------+

------------
操作系统参数
------------

EMQ X 在Linux环境下独立部署，支持10万线并发连接，需设置内核参数、TCP协议栈参数。

系统全局文件句柄
----------------

系统全局允许分配的最大文件句柄数256K:

.. code-block:: console

    # 2 millions system-wide
    sysctl -w fs.file-max=262144
    sysctl -w fs.nr_open=262144
    echo 262144 > /proc/sys/fs/nr_open

允许当前会话/进程打开文件句柄数:

.. code-block:: console

    ulimit -n 262144

/etc/sysctl.conf
----------------

持久化'fs.file-max'设置到/etc/sysctl.conf文件:

.. code-block:: console

    fs.file-max = 262144

/etc/security/limits.conf
-------------------------

/etc/security/limits.conf持久化设置允许用户/进程打开文件句柄数::

    emqx      soft   nofile      262144
    emqx      hard   nofile      262144

注: Ubuntu下需设置/etc/systemd/system.conf:

.. code-block:: properties

    DefaultLimitNOFILE=262144

--------------
EMQ X 节点名称
--------------

设置节点名称与Cookie(集群节点间通信认证)。

emqx1节点/etc/emqx/emqx.conf文件::

    node.name   = emqx1@192.168.0.10
    node.cookie = secret_dist_cookie

emqx2节点/etc/emqx/emqx.conf文件::

    node.name   = emqx2@192.168.0.20
    node.cookie = secret_dist_cookie

--------------
EMQ X 节点启动
--------------

如果RPM或DEB方式安装，启动节点::

    service emqx start

如果独立zip包安装，启动节点::
    
    ./bin/emqx start

--------------
EMQ X 节点集群
--------------

启动两台节点后，emqx1@192.168.0.10上执行::

    $ ./bin/emqx_ctl cluster join emqx2@192.168.0.20

    Join the cluster successfully.
    Cluster status: [{running_nodes,['emqx1@192.168.0.10','emqx2@192.168.0.20']}]

或，emqx2@192.168.0.20上执行::

    $ ./bin/emqx_ctl cluster join emqx1@192.168.0.10

    Join the cluster successfully.
    Cluster status: [{running_nodes,['emqx1@192.168.0.10','emqx2@192.168.0.20']}]

任意节点上查询集群状态::

    $ ./bin/emqx_ctl cluster status

    Cluster status: [{running_nodes,['emqx1@192.168.0.10','emqx2@192.168.0.20']}]

--------------
Web 管理控制台
--------------

18083端口是Web管理控制占用，该端口由'emqx-dashboard'插件启用。

控制台URL: http:://localhost:18083/ ，默认登录用户名: admin, 密码: public。

用户可以通过控制台，查询集群节点、MQTT报文统计、MQTT客户端、MQTT会话与路由信息。

.. _tcp_ports:

---------------
MQTT服务TCP端口
---------------

EMQ X 默认启用的外部MQTT服务端口包括:

+-----------+-----------------------------------+
| 1883      | MQTT协议端口                      |
+-----------+-----------------------------------+
| 8883      | MQTT/SSL端口                      |
+-----------+-----------------------------------+
| 8083      | MQTT/WebSocket端口                |
+-----------+-----------------------------------+
| 8084      | MQTT/WebSocket/SSL端口            |
+-----------+-----------------------------------+
| 18083     | Web管理控制台端口                 |
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

---------------
节点集群TCP端口
---------------

EMQ X节点间防火墙必须开放下述端口:

+-----------+-----------------------------------+
| 4369      | 集群节点发现端口                  |
+-----------+-----------------------------------+
| 5369      | 集群节点数据通道                  |
+-----------+-----------------------------------+
| 6369      | 集群节点控制通道                  |
+-----------+-----------------------------------+

