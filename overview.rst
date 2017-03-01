
.. _introduce:

========
产品介绍
========

----
概述
----

*EMQ* 是目前全球市场广泛应用的百万级开源MQTT消息服务器，全球市场(西欧、北美、印度、中国)累积超5000家企业用户，产品环境下部署超10万节点，承载MQTT连接超3000万线。

EMQ企业版是基于 *EMQ* 开发设计的商业服务版本，简称 *EMQ X* 寓意我们希望把EMQ的架构和设计优化到极致。 *EMQ X* 企业版大幅改进系统设计架构，采用 *Scalable RPC* 机制，支持更稳定的节点集群与更高性能的消息路由。

*EMQ* 企业版同时支持MQTT消息数据存储Redis、MySQL、PostgreSQL、MongoDB、Cassandra多种数据库，支持桥接转发消息到Kafka、RabbitMQ企业消息中间件。

*EMQ* 企业版可以作为智能硬件、智能家居、物联网、车联网、移动消息等应用的一站式开发平台。

.. image:: _static/images/emqplus_enterprise.png

.. _scalable_rpc:

----------------
Scalable RPC架构
----------------

*EMQ X* 企业版改进了分布节点间的通信机制，分离Erlang自身的集群通道与EMQ的数据通道，大幅提高集群节点间的消息吞吐与集群稳定性:

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
    rpc.socket_keepalive_idle = 7200

    ## Seconds between probes
    rpc.socket_keepalive_interval = 75

    ## Probes lost to close the connection
    rpc.socket_keepalive_count = 9

.. NOTE:: 集群节点间如存在防火墙，必须打开5369端口。

.. _fastlane:

------------
Fastlane订阅
------------

EMQ企业版增加了快车道(Fastlane)订阅功能，大幅提高消息路由效率，非常适合数据采集类的物联网应用:

.. image:: _static/images/fastlane.png

Fastlane订阅使用方式: 主题加 *$fastlane/* 前缀。

Fastlane订阅限制:

1. CleanSession = true
2. Qos = 0

Fastlane订阅适合物联网传感器数据采集类应用::

                 -----------------
    Sensor ----> |               |
                 |     EMQ X     | --$fastlane/$queue/#--> Subscriber
                 |    Cluster    | --$fastlane/$queue/#--> Subscriber
    Sensor ----> |               |
                 -----------------

--------
代理订阅
--------

EMQ企业版支持服务端代理订阅功能，MQTT客户端上线后无需发送SUBSCRIBE请求，EMQ代理从Redis、MySQL等数据库帮客户端加载订阅。

EMQ代理订阅功能在低功耗、低带宽网络环境下，可以节省客户端到EMQ的往返报文与流量。

------------
消息数据存储
------------

EMQ企业版支持存储订阅关系、MQTT消息、设备状态到Redis、MySQL、PostgreSQL、MongoDB与Cassandra数据库:

.. imager:: _static/img/storage.png

数据存储相关配置，详见"数据存储"章节。

------------
消息数据桥接
------------

EMQ企业版支持直接转发MQTT消息到RabbitMQ、Kafka，可作为百万级的物联网接入服务器(IoT Hub):

.. image:: _static/images/iothub.png

