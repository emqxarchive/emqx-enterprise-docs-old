
.. _getstarted:

========
开始使用
========

.. _intro:

----------
*EMQ* 2.0
----------

*EMQ* (Erlang MQTT Broker)是基于Erlang/OTP平台开发的开源物联网MQTT消息服务器。Erlang/OTP是出色的软实时(Soft-Realtime)、低延时(Low-Latency)、分布式(Distributed)的语言平台。MQTT是轻量的(Lightweight)、发布订阅模式(PubSub)的物联网消息协议。

*EMQ* 项目设计目标是承载移动终端或物联网终端海量的MQTT连接，并实现在海量物联网设备间快速低延时(Low-Latency)消息路由:

1. 稳定承载大规模的MQTT客户端连接，单服务器节点支持50万到100万连接。

2. 分布式节点集群，快速低延时的消息路由，单集群支持1000万规模的路由。

3. 消息服务器内扩展，支持定制多种认证方式、高效存储消息到后端数据库。

4. 完整物联网协议支持，MQTT、MQTT-SN、CoAP、WebSocket或私有协议支持。

--------------
*EMQ* R2企业版
--------------

EMQ R2企业版是在EMQ开源版本基础上，大幅改进了节点集群与消息路由设计，支持消息数据存储与Kafka桥接:

1. Scalable RPC架构: 分离Erlang自身的集群通道与EMQ的数据通道，大幅提高集群节点的消息吞吐与集群稳定性。

2. Fastlane订阅: 专为数据采集型物联网应用提供的Fastlane快速消息路由。

3. Redis存储订阅关系、设备在线状态、MQTT消息、保留消息，发布SUB/UNSUB事件。

4. MySQL存储订阅关系、设备在线状态、MQTT消息、保留消息。
   
5. PostgreSQL存储订阅关系、设备在线状态、MQTT消息、保留消息。
 
6. MongoDB存储订阅关系、设备在线状态、MQTT消息、保留消息。

7. Kafka桥接：EMQ内置Bridge直接转发MQTT消息到Kafka。

.. _mqtt_pubsub:

--------------------
MQTT发布订阅模式简述
--------------------

MQTT是发布订阅(Publish/Subscribe)模式的消息协议，与HTTP协议请求响应(Request/Response)模式不同。

MQTT发布者与订阅者之间通过"主题"(Topic)进行消息路由，主题(Topic)格式类似Unix文件路径，例如::

    sensor/1/temperature

    chat/room/subject

    presence/user/feng

    sensor/1/#

    sensor/+/temperature

    uber/drivers/joe/inbox

MQTT主题(Topic)支持'+', '#'的通配符，'+'通配一个层级，'#'通配多个层级(必须在末尾)。

MQTT消息发布者(Publisher)只能向特定'名称主题'(不支持通配符)发布消息，订阅者(Subscriber)通过订阅'过滤主题'(支持通配符)来匹配消息。

.. NOTE::

    初接触MQTT协议的用户，通常会向通配符的'过滤主题'发布广播消息，MQTT协议不支持这种模式，需从订阅侧设计广播主题(Topic)。
    例如Android推送，向所有广州用户，推送某类本地消息，客户端获得GIS位置后，可订阅'news/city/guangzhou'主题。

.. _quick_start:

------------
安装启动EMQX
------------

EMQX支持Ubuntu、CentOS、FreeBSD、Mac OS X、Windows平台程序包与Docker镜像。

EMQX安装包名格式例如: emqx-enterprise-centos7-r2.1.0.zip

获取程序包可直接解压启动运行，例如CentOS7平台:

.. code-block:: bash

    unzip emqx-enterprise-centos7-r2.1.0.zip && cd emqx

    # 启动
    ./bin/emqx start

    # 检查运行状态
    ./bin/emqx_ctl status

    # 停止
    ./bin/emqx stop

EMQX默认允许匿名认证，启动后MQTT客户端可连接1883端口，启动运行日志输出在log/目录。

.. _compile:

.. _dashboard:

------------------------
Web管理控制台(Dashboard)
------------------------

EMQX企业版服务器启动后，会默认加载Dashboard插件，启动Web管理控制台。用户可通过Web控制台，查看服务器运行状态、统计数据、客户端(Client)、会话(Session)、主题(Topic)、订阅(Subscription)、插件(Plugin)。

控制台地址: http://127.0.0.1:18083，默认用户: admin，密码：public

.. image:: ./_static/images/dashboard.png

.. _features:

------------------
EMQX企业版功能列表
------------------

* 完整的MQTT V3.1/V3.1.1协议规范支持
* QoS0, QoS1, QoS2消息支持
* 持久会话与离线消息支持
* Retained消息支持
* Last Will消息支持
* TCP/SSL连接支持
* MQTT/WebSocket(SSL)支持
* HTTP消息发布接口支持
* $SYS/#系统主题支持
* 客户端在线状态查询与订阅支持
* 客户端ID或IP地址认证支持
* 用户名密码认证支持
* LDAP认证
* Redis、MySQL、PostgreSQL、MongoDB、HTTP认证集成
* 浏览器Cookie认证
* 基于客户端ID、IP地址、用户名的访问控制(ACL)
* 多服务器节点集群(Cluster)
* 多服务器节点桥接(Bridge)
* mosquitto桥接支持
* Stomp协议支持
* MQTT-SN协议支持
* CoAP协议支持
* Stomp/SockJS支持
* 通过Paho兼容性测试
* 本地订阅($local/topic)
* 共享订阅($share/<group>/topic)
* sysctl类似k = v格式配置文件
* Redis数据存储
* MySQL数据存储
* PostgreSQL数据存储
* MongoDB数据存储
* Kafka桥接

.. _plugins:

-----------------
EMQX 扩展插件列表
-----------------

EMQX企业版支持丰富的扩展插件，包括控制台、扩展模块、认证方式、多种接入协议、数据存储、Kafka桥接:

+----------------------------+-----------------------------------+
| `emqx_retainer`_           | Retain消息存储模块                |
+----------------------------+-----------------------------------+
| `emqx_modules`_            | Presence, Subscription扩展模块    |
+----------------------------+-----------------------------------+
| `emqx_dashboard`_          | Web管理控制台，默认加载           |
+----------------------------+-----------------------------------+
| `emqx_auth_clientid`_      | ClientId、密码认证插件            |
+----------------------------+-----------------------------------+
| `emqx_auth_username`_      | 用户名、密码认证插件              |
+----------------------------+-----------------------------------+
| `emqx_auth_ldap`_          | LDAP认证插件                      |
+----------------------------+-----------------------------------+
| `emqx_auth_http`_          | HTTP认证插件                      |
+----------------------------+-----------------------------------+
| `emqx_auth_mysql`_         | MySQL认证插件                     |
+----------------------------+-----------------------------------+
| `emqx_auth_pgsql`_         | PostgreSQL认证插件                |
+----------------------------+-----------------------------------+
| `emqx_auth_redis`_         | Redis认证插件                     |
+----------------------------+-----------------------------------+
| `emqx_auth_mongo`_         | MongoDB认证插件                   |
+----------------------------+-----------------------------------+
| `emqx_stomp`_              | Stomp协议插件                     |
+----------------------------+-----------------------------------+
| `emqx_recon`_              | Recon优化调测插件                 |
+----------------------------+-----------------------------------+
| `emqx_reloader`_           | 热升级插件(开发调试)              |
+----------------------------+-----------------------------------+
| `emqx_backend_redis`_      | Redis数据存储                     |
+----------------------------+-----------------------------------+
| `emqx_backend_mysql`_      | MySQL数据存储                     |
+----------------------------+-----------------------------------+
| `emqx_backend_pgsql`_      | PostgreSQL数据存储                |
+----------------------------+-----------------------------------+
| `emqx_backend_mongo`_      | MongoDB数据存储                   |
+----------------------------+-----------------------------------+
| `emqx_bridge_kafka`_       | Kafka桥接                         |
+----------------------------+-----------------------------------+

扩展插件通过'bin/emqx_ctl'管理命令行，或Dashboard控制台加载启用。例如启用PostgreSQL认证插件::

    ./bin/emqx_ctl plugins load emqx_auth_pgsql

Linux操作系统参数
-----------------

# 2M - 系统所有进程可打开的文件数量::

    sysctl -w fs.file-max=2097152
    sysctl -w fs.nr_open=2097152

# 1M - 系统允许当前进程打开的文件数量::

    ulimit -n 1048576

TCP协议栈参数
-------------

# backlog - Socket监听队列长度::

    sysctl -w net.core.somaxconn=65536

Erlang虚拟机参数
----------------

emqx/etc/emqx.conf:

.. code-block:: properties

    ## Erlang Process Limit
    node.process_limit = 2097152

    ## Sets the maximum number of simultaneously existing ports for this system
    node.max_ports = 1048576

EMQX最大允许连接数
------------------

emqttd/etc/emq.conf 'listeners'段落::

    ## Size of acceptor pool
    mqtt.listener.tcp.acceptors = 64

    ## Maximum number of concurrent clients
    mqtt.listener.tcp.max_clients = 1000000

测试客户端设置
--------------

测试客户端在一个接口上，最多只能创建65000连接::

    sysctl -w net.ipv4.ip_local_port_range="500 65535"

    echo 1000000 > /proc/sys/fs/nr_open

按应用场景测试
--------------

MQTT是一个设计得非常出色的传输层协议，在移动消息、物联网、车联网、智能硬件甚至能源勘探等领域有着广泛的应用。1个字节报头、2个字节心跳、消息QoS支持等设计，非常适合在低带宽、不可靠网络、嵌入式设备上应用。

不同的应用有不同的系统要求，用户使用emqttd消息服务器前，可以按自己的应用场景进行测试，而不是简单的连接压力测试:

1. Android消息推送: 推送消息广播测试。

2. 移动即时消息应用: 消息收发确认测试。

3. 智能硬件应用: 消息的往返时延测试。

4. 物联网数据采集: 并发连接与吞吐测试。

.. _mqtt_clients:

------------------
开源MQTT客户端项目
------------------

GitHub: https://github.com/emqtt

+--------------------+----------------------+
| `emqttc`_          | Erlang MQTT客户端库  |
+--------------------+----------------------+
| `emqtt_benchmark`_ | MQTT连接测试工具     |
+--------------------+----------------------+
| `CocoaMQTT`_       | Swift语言MQTT客户端库|
+--------------------+----------------------+
| `QMQTT`_           | QT框架MQTT客户端库   |
+--------------------+----------------------+

Eclipse Paho: https://www.eclipse.org/paho/

MQTT.org: https://github.com/mqtt/mqtt.github.io/wiki/libraries

.. _emqttc:          https://github.com/emqtt/emqttc
.. _emqtt_benchmark: https://github.com/emqtt/emqtt_benchmark
.. _CocoaMQTT:       https://github.com/emqtt/CocoaMQTT
.. _QMQTT:           https://github.com/emqtt/qmqtt

