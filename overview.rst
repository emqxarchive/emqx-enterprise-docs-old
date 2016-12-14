
.. _introduce:

========
产品介绍
========

----
概述
----

*EMQ* 是目前全球市场广泛应用的开源MQTT消息服务器，全球市场(西欧、北美、印度、中国)累积超5000家企业用户，产品环境下承载超2000万线MQTT连接。EMQPLUS企业版是基于 *EMQ* 开发设计的商业服务版本， EMQPLUS企业版支持更稳定的节点集群与更高性能的消息路由，支持消息数据存储Redis、MySQL、PostgreSQL与MongoDB等多种数据库，并由专业团队提供技术支持与咨询服务。EMQPLUS企业版帮助客户快速开发基于MQTT协议的物联网、车联网、智能硬件和移动消息应用。

.. image:: _static/images/emqplus_enterprise.png

------
企业版
------

EMQPLUS企业版支持 *Scalable RPC* 方式节点集群与 *Fastlane订阅* 快速消息路由，支持Syslog集成与多种数据库消息存储。

.. _scalable_rpc:

Scalable RPC架构
----------------

EMQPLUS企业版改进了分布节点间的通信机制，分离Erlang自身的集群通道与EMQ的数据通道，大幅提高集群节点间的消息吞吐与集群稳定性:

.. NOTE:: 虚线为Erlang的分布集群通道，实线为节点间消息数据通道。

.. image:: _static/images/scalable_rpc.png

*Scalable RPC* 配置::

    ## TCP server port.
    rpc.tcp_server_port = 5369

    ## Default TCP port for outgoing connections
    rpc.tcp_client_port = 5369

    ## Client connect timeout
    rpc.connect_timeout = 5000

    ## Client and Server send timeout
    rpc.send_timeout = 5000

    ## Authentication timeout
    rpc.authentication_timeout = 5000

    ## Default receive timeout for call() functions
    rpc.call_receive_timeout = 15000

    ## Socket keepalive configuration
    rpc.socket_keepalive_idle = 5

    ## Seconds between probes
    rpc.socket_keepalive_interval = 5

    ## Probes lost to close the connection
    rpc.socket_keepalive_count = 2

.. NOTE:: 集群节点间如存在防火墙，必须打开5369端口。

.. _fastlane:

Fastlane订阅
------------

EMQPLUS企业版增加了Fastlane订阅功能，大幅提高消息路由效率，非常适合数据采集类的物联网应用:

.. image:: _static/images/fastlane.png

Fastlane订阅使用方式: 主题加 *$fastlane/* 前缀。

Fastlane订阅限制:

1. CleanSession = true
2. Qos = 0

Fastlane订阅适合物联网传感器数据采集类应用::

                  -----------------
    Sensor ---->  |               |
                  |      EMQ      | --$fastlane/$queue/#--> Subscriber
                  |    Cluster    | --$fastlane/$queue/#--> Subscriber
    Sensor ---->  |               |
                  -----------------

Syslog日志
----------

EMQPLUS企业版支持输出日志到Syslog，Syslog参数通过etc/emq.conf配置:

.. code-block:: properties

    ## Syslog. Enum: on, off
    log.syslog = on 

    ##  syslog level. Enum: debug, info, notice, warning, error, critical, alert, emergency
    log.syslog.level = error

数据存储
--------

EMQPLUS企业版支持存储订阅关系、MQTT消息、设备状态到Redis、MySQL、PostgreSQL、MongoDB数据库。

数据存储相关配置，详见"数据存储"章节。

