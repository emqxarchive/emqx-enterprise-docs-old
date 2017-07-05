
.. _guide:

========
用户指南
========

------------
MQTT发布订阅
------------

MQTT是为移动互联网、物联网设计的轻量发布订阅模式的消息服务器:

.. image:: ./_static/images/guide_1.png

EMQ X服务器安装启动后，任何设备或终端的MQTT客户端，可通过MQTT协议连接到服务器，发布订阅消息方式互通。

MQTT协议客户端库: https://github.com/mqtt/mqtt.github.io/wiki/libraries

例如，mosquitto_sub/pub命令行发布订阅消息::

    mosquitto_sub -t topic -q 2
    mosquitto_pub -t topic -q 1 -m "Hello, MQTT!"

MQTT V3.1.1版本协议规范: http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/mqtt-v3.1.1.html

EMQ X消息服务器的MQTT协议TCP监听器，可在emqx.conf文件中设置:

.. code-block:: properties

    ## TCP Listener: 1883, 127.0.0.1:1883, ::1:1883
    mqtt.listener.tcp.external = 1883

    ## Size of acceptor pool
    mqtt.listener.tcp.external.acceptors = 8

    ## Maximum number of concurrent clients
    mqtt.listener.tcp.external.max_clients = 1024

MQTT(SSL) TCP监听器，缺省端口8883:

.. code-block:: properties

    ## SSL Listener: 8883, 127.0.0.1:8883, ::1:8883
    mqtt.listener.ssl.external = 8883

    ## Size of acceptor pool
    mqtt.listener.ssl.external.acceptors = 4

    ## Maximum number of concurrent clients
    mqtt.listener.ssl.external.max_clients = 512

.. _shared_subscription:

--------
共享订阅
--------

共享订阅(Shared Subscription)支持在多订阅者间采用分组负载平衡方式派发消息::

                                ---------
                                |       | --Msg1--> Subscriber1
    Publisher--Msg1,Msg2,Msg3-->| EMQ X | --Msg2--> Subscriber2
                                |       | --Msg3--> Subscriber3
                                ---------

共享订阅支持两种使用方式:

+-----------------+-------------------------------------------+
|  订阅前缀       | 使用示例                                  |
+-----------------+-------------------------------------------+
| $queue/         | mosquitto_sub -t '$queue/topic'           |
+-----------------+-------------------------------------------+
| $share/<group>/ | mosquitto_sub -t '$share/group/topic'     |
+-----------------+-------------------------------------------+

.. _local_subscription:

--------
本地订阅
--------

本地订阅(Local Subscription)只在本节点创建订阅与路由表，不会在集群节点间广播全局路由:

.. code-block:: shell

    mosquitto_sub -t '$local/topic'

    mosquitto_pub -t 'topic'

使用方式: 订阅者在主题(Topic)前增加'$local/'前缀。

.. _fastlane_subscription:

------------
Fastlane订阅
------------

EMQ X企业版支持的快车道(Fastlane)订阅功能，提高到订阅者的消息派发效率:

.. image:: _static/images/guide_3.png

Fastlane订阅使用方式: 主题加 *$fastlane/* 前缀。

Fastlane订阅限制:

1. CleanSession = true
2. Qos = 0

Fastlane订阅适合物联网传感器数据采集类应用:

.. image:: _static/images/guide_4.png

.. _http_publish:

------------
HTTP发布接口
------------

EMQ X提供了一个HTTP发布接口，应用服务器或Web服务器可通过该接口发布MQTT消息::

    HTTP POST http://host:8083/mqtt/publish

Web服务器例如PHP/Java/Python/NodeJS或Ruby on Rails，可通过HTTP POST请求发布MQTT消息:

.. code-block:: bash

    curl -v --basic -u user:passwd -d "qos=1&retain=0&topic=/a/b/c&message=hello from http..." -k http://localhost:8083/mqtt/publish

HTTP接口参数:

+---------+----------------+
| 参数    | 说明           |
+=========+================+
| client  | MQTT客户端ID   |
+---------+----------------+
| qos     | QoS: 0 | 1 | 2 |
+---------+----------------+
| retain  | Retain: 0 | 1  |
+---------+----------------+
| topic   | 主题(Topic)    |
+---------+----------------+
| message | 消息           |
+---------+----------------+

.. NOTE:: HTTP接口采用Basic认证

------------------
MQTT WebSocket连接
------------------

EMQ X服务器支持MQTT WebSocket连接，Web浏览器可直接通过MQTT协议连接服务器:

+-------------------------+----------------------------+
| WebSocket URI:          | ws(s)://host:8083/mqtt     |
+-------------------------+----------------------------+
| Sec-WebSocket-Protocol: | 'mqttv3.1' or 'mqttv3.1.1' |
+-------------------------+----------------------------+

Dashboard插件提供了一个MQTT WebSocket连接的测试页面::

    http://127.0.0.1:18083/websocket.html

EMQ X通过内嵌的HTTP服务器，实现MQTT WebSocket与HTTP发布接口，etc/emqx.conf设置:

.. code-block:: properties

    ## HTTP and WebSocket Listener
    mqtt.listener.http.external = 8083
    mqtt.listener.http.external.acceptors = 4
    mqtt.listener.http.external.max_clients = 64

.. _sys_topic:

-------------
$SYS-系统主题
-------------

EMQ X服务器周期性发布自身运行状态、MQTT协议统计、客户端上下线状态到'$SYS/'开头系统主题。

$SYS主题路径以"$SYS/brokers/{node}/"开头，'${node}'是Erlang节点名称::

    $SYS/brokers/emqx@127.0.0.1/version

    $SYS/brokers/emqx@host2/uptime

.. NOTE:: 默认只允许localhost的MQTT客户端订阅$SYS主题，可通过etc/acl.config修改访问控制规则。

$SYS系统消息发布周期，通过etc/emq.conf配置:

.. code-block:: properties

    ## System Interval of publishing broker $SYS Messages
    mqtt.broker.sys_interval = 60

.. _sys_brokers:

服务器版本、启动时间与描述消息
------------------------------

+--------------------------------+-----------------------+
| 主题                           | 说明                  |
+================================+=======================+
| $SYS/brokers                   | 集群节点列表          |
+--------------------------------+-----------------------+
| $SYS/brokers/${node}/version   | EMQ X版本             |
+--------------------------------+-----------------------+
| $SYS/brokers/${node}/uptime    | EMQ X启动时间         |
+--------------------------------+-----------------------+
| $SYS/brokers/${node}/datetime  | EMQ X服务器时间       |
+--------------------------------+-----------------------+
| $SYS/brokers/${node}/sysdescr  | EMQ X版本描述         |
+--------------------------------+-----------------------+

.. _sys_clients:

MQTT客户端上下线状态消息
------------------------

$SYS主题前缀: $SYS/brokers/${node}/clients/

+--------------------------+--------------------------------------------+------------------------------------+
| 主题(Topic)              | 数据(JSON)                                 | 说明                               |
+==========================+============================================+====================================+
| ${clientid}/connected    | {ipaddress: "127.0.0.1", username: "test", | Publish when a client connected    |
|                          |  session: false, version: 3, connack: 0,   |                                    |
|                          |  ts: 1432648482}                           |                                    |
+--------------------------+--------------------------------------------+------------------------------------+
| ${clientid}/disconnected | {reason: "keepalive_timeout",              | Publish when a client disconnected |
|                          |  ts: 1432749431}                           |                                    |
+--------------------------+--------------------------------------------+------------------------------------+

'connected'消息JSON数据:

.. code-block:: json

    {
        ipaddress: "127.0.0.1",
        username:  "test",
        session:   false,
        protocol:  3,
        connack:   0,
        ts:        1432648482
    }

'disconnected'消息JSON数据:

.. code-block:: json

    {
        reason: normal,
        ts:     1432648486
    }

.. _sys_stats:

Statistics - 系统统计消息
--------------------------

系统主题前缀: $SYS/brokers/${node}/stats/

Clients - 客户端统计
....................

+---------------------+---------------------------------------------+
| 主题(Topic)         | 说明                                        |
+---------------------+---------------------------------------------+
| clients/count       | 当前客户端总数                              |
+---------------------+---------------------------------------------+
| clients/max         | 最大客户端数量                              |
+---------------------+---------------------------------------------+

Sessions - 会话统计
...................

+---------------------+---------------------------------------------+
| 主题(Topic)         | 说明                                        |
+---------------------+---------------------------------------------+
| sessions/count      | 当前会话总数                                |
+---------------------+---------------------------------------------+
| sessions/max        | 最大会话数量                                |
+---------------------+---------------------------------------------+

Subscriptions - 订阅统计
........................

+---------------------+---------------------------------------------+
| 主题(Topic)         | 说明                                        |
+---------------------+---------------------------------------------+
| subscriptions/count | 当前订阅总数                                |
+---------------------+---------------------------------------------+
| subscriptions/max   | 最大订阅数量                                |
+---------------------+---------------------------------------------+

Topics - 主题统计
.................

+---------------------+---------------------------------------------+
| 主题(Topic)         | 说明                                        |
+---------------------+---------------------------------------------+
| topics/count        | 当前Topic总数(跨节点)                       |
+---------------------+---------------------------------------------+
| topics/max          | Max number of topics                        |
+---------------------+---------------------------------------------+

Metrics-收发流量/报文/消息统计
------------------------------

系统主题(Topic)前缀: $SYS/brokers/${node}/metrics/

收发流量统计
............

+---------------------+---------------------------------------------+
| 主题(Topic)         | 说明                                        |
+---------------------+---------------------------------------------+
| bytes/received      | 累计接收流量                                |
+---------------------+---------------------------------------------+
| bytes/sent          | 累计发送流量                                |
+---------------------+---------------------------------------------+

MQTT报文收发统计
................

+--------------------------+---------------------------------------------+
| 主题(Topic)              | 说明                                        |
+--------------------------+---------------------------------------------+
| packets/received         | 累计接收MQTT报文                            |
+--------------------------+---------------------------------------------+
| packets/sent             | 累计发送MQTT报文                            |
+--------------------------+---------------------------------------------+
| packets/connect          | 累计接收MQTT CONNECT报文                    |
+--------------------------+---------------------------------------------+
| packets/connack          | 累计发送MQTT CONNACK报文                    |
+--------------------------+---------------------------------------------+
| packets/publish/received | 累计接收MQTT PUBLISH报文                    |
+--------------------------+---------------------------------------------+
| packets/publish/sent     | 累计发送MQTT PUBLISH报文                    |
+--------------------------+---------------------------------------------+
| packets/subscribe        | 累计接收MQTT SUBSCRIBE报文                  |
+--------------------------+---------------------------------------------+
| packets/suback           | 累计发送MQTT SUBACK报文                     |
+--------------------------+---------------------------------------------+
| packets/unsubscribe      | 累计接收MQTT UNSUBSCRIBE报文                |
+--------------------------+---------------------------------------------+
| packets/unsuback         | 累计发送MQTT UNSUBACK报文                   |
+--------------------------+---------------------------------------------+
| packets/pingreq          | 累计接收MQTT PINGREQ报文                    |
+--------------------------+---------------------------------------------+
| packets/pingresp         | 累计发送MQTT PINGRESP报文数量               |
+--------------------------+---------------------------------------------+
| packets/disconnect       | 累计接收MQTT DISCONNECT数量                 |
+--------------------------+---------------------------------------------+

MQTT消息收发统计
................

+--------------------------+---------------------------------------------+
| 主题(Topic)              | 说明                                        |
+--------------------------+---------------------------------------------+
| messages/received        | 累计接收消息                                |
+--------------------------+---------------------------------------------+
| messages/sent            | 累计发送消息                                |
+--------------------------+---------------------------------------------+
| messages/retained        | Retained消息总数                            |
+--------------------------+---------------------------------------------+
| messages/dropped         | 丢弃消息总数                                |
+--------------------------+---------------------------------------------+

.. _sys_alarms:

Alarms-系统告警
---------------

系统主题(Topic)前缀: $SYS/brokers/${node}/alarms/

+------------------+------------------+
| 主题(Topic)      | 说明             |
+------------------+------------------+
| ${alarmId}/alert | 新产生告警       |
+------------------+------------------+
| ${alarmId}/clear | 清除告警         |
+------------------+------------------+

.. _sys_sysmon:

Sysmon-系统监控
---------------

系统主题(Topic)前缀: $SYS/brokers/${node}/sysmon/

+------------------+--------------------+
| 主题(Topic)      | 说明               |
+------------------+--------------------+
| long_gc          | GC时间过长警告     |
+------------------+--------------------+
| long_schedule    | 调度时间过长警告   |
+------------------+--------------------+
| large_heap       | Heap内存占用警告   |
+------------------+--------------------+
| busy_port        | Port忙警告         |
+------------------+--------------------+
| busy_dist_port   | Dist Port忙警告    |
+------------------+--------------------+

.. _trace:

----
追踪
----

EMQ X支持追踪来自某个客户端(Client)的全部报文，或者发布到某个主题(Topic)的全部消息。

追踪客户端(Client):

.. code-block:: bash

    ./bin/emqx_ctl trace client "clientid" "trace_clientid.log"

追踪主题(Topic):

.. code-block:: bash

    ./bin/emqx_ctl trace topic "topic" "trace_topic.log"

查询追踪:

.. code-block:: bash

    ./bin/emqx_ctl trace list

停止追踪:

.. code-block:: bash

    ./bin/emqx_ctl trace client "clientid" off

    ./bin/emqx_ctl trace topic "topic" off

