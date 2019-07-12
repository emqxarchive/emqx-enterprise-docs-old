
.. _guide:

用户指南 (User Guide)
^^^^^^^^^^^^^^^^^^^^^^

.. _start:

程序启动
---------

下载地址: https://www.emqx.io/downloads/broker?osType=Linux

程序包下载后，可直接解压启动运行，例如 macOS 平台:

.. code-block:: bash

    unzip emqx-macosx-v3.1.0.zip && cd emqx

    # 启动emqx
    ./bin/emqx start

    # 检查运行状态
    ./bin/emqx_ctl status

*EMQ X* 消息服务器默认占用的 TCP 端口包括:

+-----------+-----------------------------------+
| 1883      | MQTT 协议端口                     |
+-----------+-----------------------------------+
| 8883      | MQTT/SSL 端口                     |
+-----------+-----------------------------------+
| 8083      | MQTT/WebSocket 端口               |
+-----------+-----------------------------------+
| 8080      | HTTP API 端口                     |
+-----------+-----------------------------------+
| 18083     | Dashboard 管理控制台端口          |
+-----------+-----------------------------------+

.. _pubsub:

MQTT 发布订阅
-------------

MQTT 是为移动互联网、物联网设计的轻量发布订阅模式的消息服务器，目前支持 MQTT `v3.1.1 <http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/mqtt-v3.1.1.html>`_ 和 `v5.0 <http://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html>`_:

.. image:: ./_static/images/pubsub_concept.png

*EMQ X* 启动后，任何设备或终端可通过 MQTT 协议连接到服务器，通过 **发布(Publish)/订阅(Subscribe)** 进行交换消息。

MQTT 客户端库: https://github.com/mqtt/mqtt.github.io/wiki/libraries

例如，mosquitto_sub/pub 命令行发布订阅消息::

    mosquitto_sub -h 127.0.0.1 -p 1883 -t topic -q 2
    mosquitto_pub -h 127.0.0.1 -p 1883 -t topic -q 1 -m "Hello, MQTT!"

.. _authentication:

认证/访问控制
-------------

**EMQ X** 消息服务器 *连接认证* 和 *访问控制* 由一系列的认证插件(Plugins)提供，他们的命名都符合 ``emqx_auth_<name>`` 的规则。

在 EMQ X 中，这两个功能分别是指：

1. **连接认证**: *EMQ X* 校验每个连接上的客户端是否具有接入系统的权限，若没有则会断开该连接
2. **访问控制**: *EMQ X* 校验客户端每个 *发布(Publish)/订阅(Subscribe)* 的权限，以 *允许/拒绝* 相应操作

认证(Authentication)
>>>>>>>>>>>>>>>>>>>>>

*EMQ X* 消息服务器认证由一系列认证插件(Plugins)提供，系统支持按用户名密码、ClientID 或匿名认证。

系统默认开启匿名认证(Anonymous)，通过加载认证插件可开启的多个认证模块组成认证链::

               ----------------           ----------------           ------------
    Client --> | Username认证 | -ignore-> | ClientID认证 | -ignore-> | 匿名认证 |
               ----------------           ----------------           ------------
                      |                         |                         |
                     \|/                       \|/                       \|/
                allow | deny              allow | deny              allow | deny

**开启匿名认证**

etc/emqx.conf 配置启用匿名认证:

.. code:: properties

    允许匿名访问
    ## Value: true | false
    allow_anonymous = true

.. _acl:

访问控制(ACL)
>>>>>>>>>>>>>

*EMQ X* 消息服务器通过 ACL(Access Control List) 实现 MQTT 客户端访问控制。

ACL 访问控制规则定义::

    允许(Allow)|拒绝(Deny) 谁(Who) 订阅(Subscribe)|发布(Publish) 主题列表(Topics)

MQTT 客户端发起订阅/发布请求时，EMQ X 消息服务器的访问控制模块会逐条匹配 ACL 规则，直到匹配成功为止::

              ---------              ---------              ---------
    Client -> | Rule1 | --nomatch--> | Rule2 | --nomatch--> | Rule3 | --> Default
              ---------              ---------              ---------
                  |                      |                      |
                match                  match                  match
                 \|/                    \|/                    \|/
            allow | deny           allow | deny           allow | deny

**默认访问控制设置**

*EMQ X* 消息服务器默认访问控制，在 etc/emqx.conf 中设置:

.. code:: properties

    ## 设置所有 ACL 规则都不能匹配时是否允许访问
    ## Value: allow | deny
    acl_nomatch = allow

    ## 设置存储 ACL 规则的默认文件
    ## Value: File Name
    acl_file = etc/acl.conf

ACL 规则定义在 etc/acl.conf，EMQ X 启动时加载到内存:

.. code:: erlang

    %% 允许 'dashboard' 用户订阅 '$SYS/#'
    {allow, {user, "dashboard"}, subscribe, ["$SYS/#"]}.

    %% 允许本机用户发布订阅全部主题
    {allow, {ipaddr, "127.0.0.1"}, pubsub, ["$SYS/#", "#"]}.

    %% 拒绝除本机用户以外的其他用户订阅 '$SYS/#' 与 '#' 主题
    {deny, all, subscribe, ["$SYS/#", {eq, "#"}]}.

    %% 允许上述规则以外的任何情形
    {allow, all}.


EMQ X 提供的认证插件包括:

+----------------------------+---------------------------+
| 插件                       | 说明                      |
+============================+===========================+
| `emqx_auth_clientid`_      | ClientId 认证/鉴权插件    |
+----------------------------+---------------------------+
| `emqx_auth_username`_      | 用户名密码认证/鉴权插件   |
+----------------------------+---------------------------+
| `emqx_auth_jwt`_           | JWT 认证/鉴权插件         |
+----------------------------+---------------------------+
| `emqx_auth_ldap`_          | LDAP 认证/鉴权插件        |
+----------------------------+---------------------------+
| `emqx_auth_http`_          | HTTP 认证/鉴权插件        |
+----------------------------+---------------------------+
| `emqx_auth_mysql`_         | MySQ L认证/鉴权插件       |
+----------------------------+---------------------------+
| `emqx_auth_pgsql`_         | Postgre 认证/鉴权插件     |
+----------------------------+---------------------------+
| `emqx_auth_redis`_         | Redis 认证/鉴权插件       |
+----------------------------+---------------------------+
| `emqx_auth_mongo`_         | MongoDB 认证/鉴权插件     |
+----------------------------+---------------------------+

其中，关于每个认证插件的配置及用法，可参考 `扩展插件 (Plugins) <https://developer.emqx.io/docs/emq/v3/cn/plugins.html>`_ 关于认证部分。


.. note:: auth 插件可以同时启动多个。每次检查的时候，按照优先级从高到低依次检查，同一优先级的，先启动的插件先检查。

此外 *EMQ X* 还支持使用 **PSK (Pre-shared Key)** 的方式来控制客户端的接入，但它并不是使用的上述的 *连接认证* 链的方式，而是在 SSL 握手期间进行验证。详情参考 `Pre-shared Key <https://en.wikipedia.org/wiki/Pre-shared_key>`_ 和 `emqx_psk_file`_

.. _shared_sub:

共享订阅 (Shared Subscription)
-------------------------------

*EMQ X* R3.0 版本开始支持集群级别的共享订阅功能。共享订阅(Shared Subscription)支持多种消息派发策略::

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

示例::

    mosquitto_sub -t '$share/group/topic'

    mosquitto_pub -t 'topic' -m msg -q 2


*EMQ X* 通过 `etc/emqx.conf` 中的 `broker.shared_subscription_strategy` 字段配置共享消息的派发策略。

目前支持按以下几种策略派发消息：

+---------------------------+-------------------------+
| 策略                      | 说明                    |
+===========================+=========================+
| random                    | 在所有共享订阅者中随机  |
+---------------------------+-------------------------+
| round_robin               | 按订阅顺序              |
+---------------------------+-------------------------+
| sticky                    | 使用上次派发的订阅者    |
+---------------------------+-------------------------+
| hash                      | 根据发送者的 ClientId   |
+---------------------------+-------------------------+

.. note:: 当所有的订阅者都不在线时，仍会挑选一个订阅者，并存至其 Session 的消息队列中

.. _http_publish:

HTTP 发布接口
-------------

*EMQ X* 消息服务器提供了一个 HTTP 发布接口，应用服务器或 Web 服务器可通过该接口发布 MQTT 消息::

    HTTP POST http://host:8080/api/v3/mqtt/publish

Web 服务器例如 PHP/Java/Python/NodeJS 或 Ruby on Rails，可通过 HTTP POST 请求发布 MQTT 消息:

.. code:: bash

    curl -v --basic -u user:passwd -H "Content-Type: application/json" -d \
    '{"qos":1, "retain": false, "topic":"world", "payload":"test" , "client_id": "C_1492145414740"}' \-k http://localhost:8080/api/v3/mqtt/publish

HTTP 接口参数:

+----------+----------------------+
| 参数     | 说明                 |
+==========+======================+
| client_id| MQTT 客户端 ID       |
+----------+----------------------+
| qos      | QoS: 0 | 1 | 2       |
+----------+----------------------+
| retain   | Retain: true | false |
+----------+----------------------+
| topic    | 主题(Topic)          |
+----------+----------------------+
| payload  | 消息载荷             |
+----------+----------------------+

.. NOTE::

    HTTP 发布接口采用 `Basic <https://en.wikipedia.org/wiki/Basic_access_authentication>`_ 认证。上例中的 ``user`` 和 ``password`` 是来自于 Dashboard 下的 Applications 内的 AppId 和密码

MQTT WebSocket 连接
-------------------

*EMQ X* 还支持 WebSocket 连接，Web 浏览器可直接通过 WebSocket 连接至服务器:

+-------------------------+----------------------------+
| WebSocket URI:          | ws(s)://host:8083/mqtt     |
+-------------------------+----------------------------+
| Sec-WebSocket-Protocol: | 'mqttv3.1' or 'mqttv3.1.1' |
+-------------------------+----------------------------+

Dashboard 插件提供了一个 MQTT WebSocket 连接的测试页面::

    http://127.0.0.1:18083/#/websocket

.. _sys_topic:

$SYS-系统主题
-------------

*EMQ X* 消息服务器周期性发布自身运行状态、消息统计、客户端上下线事件到 以 ``$SYS/`` 开头系统主题。

$SYS 主题路径以 ``$SYS/brokers/{node}/`` 开头。 ``{node}`` 是指产生该 事件/消息 所在的节点名称，例如::

    $SYS/brokers/emqx@127.0.0.1/version

    $SYS/brokers/emqx@127.0.0.1/uptime

.. NOTE:: 默认只允许 localhost 的 MQTT 客户端订阅 $SYS 主题，可通过 etc/acl.config 修改访问控制规则。

$SYS 系统消息发布周期，通过 etc/emqx.conf 配置:

.. code:: properties

    ## System interval of publishing $SYS messages.
    ##
    ## Value: Duration
    ## Default: 1m, 1 minute
    broker.sys_interval = 1m

.. _sys_brokers:

集群状态信息
>>>>>>>>>>>>

+--------------------------------+-----------------------+
| 主题                           | 说明                  |
+================================+=======================+
| $SYS/brokers                   | 集群节点列表          |
+--------------------------------+-----------------------+
| $SYS/brokers/${node}/version   | EMQ X 服务器版本      |
+--------------------------------+-----------------------+
| $SYS/brokers/${node}/uptime    | EMQ X 服务器启动时间  |
+--------------------------------+-----------------------+
| $SYS/brokers/${node}/datetime  | EMQ X 服务器时间      |
+--------------------------------+-----------------------+
| $SYS/brokers/${node}/sysdescr  | EMQ X 服务器描述      |
+--------------------------------+-----------------------+

.. _sys_clients:

客户端上下线事件
>>>>>>>>>>>>>>>>

$SYS 主题前缀: $SYS/brokers/${node}/clients/

+--------------------------+------------------------------------------+
| 主题(Topic)              | 说明                                     |
+==========================+==========================================+
| ${clientid}/connected    | 上线事件。当某客户端上线时，会发布该消息 |
+--------------------------+------------------------------------------+
| ${clientid}/disconnected | 下线事件。当某客户端离线时，会发布该消息 |
+--------------------------+------------------------------------------+

'connected' 事件消息的 Payload 可解析成 JSON 格式:

.. code:: json

    {
        "clientid":"id1",
        "username":"u",
        "ipaddress":"127.0.0.1",
        "connack":0,
        "ts":1554047291,
        "proto_ver":3,
        "proto_name":"MQIsdp",
        "clean_start":true,
        "keepalive":60
    }


'disconnected' 事件消息的 Payload 可解析成 JSON 格式:

.. code:: json

    {
        "clientid":"id1",
        "username":"u",
        "reason":"normal",
        "ts":1554047291
    }

.. _sys_stats:

系统统计(Statistics)
>>>>>>>>>>>>>>>>>>>>

系统主题前缀: $SYS/brokers/${node}/stats/

客户端统计
::::::::::

+---------------------+---------------------------------------------+
| 主题(Topic)         | 说明                                        |
+---------------------+---------------------------------------------+
| connections/count   | 当前客户端总数                              |
+---------------------+---------------------------------------------+
| connections/max     | 最大客户端数量                              |
+---------------------+---------------------------------------------+

会话统计
::::::::

+-----------------------------+---------------------------------------------+
| 主题(Topic)                 | 说明                                        |
+-----------------------------+---------------------------------------------+
| sessions/count              | 当前会话总数                                |
+-----------------------------+---------------------------------------------+
| sessions/max                | 最大会话数量                                |
+-----------------------------+---------------------------------------------+
| sessions/persistent/count   | 当前持久会话总数                            |
+-----------------------------+---------------------------------------------+
| sessions/persistent/max     | 最大持久会话数量                            |
+-----------------------------+---------------------------------------------+

订阅统计
::::::::

+---------------------------------+---------------------------------------------+
| 主题(Topic)                     | 说明                                        |
+---------------------------------+---------------------------------------------+
| suboptions/count                | 当前订阅选项个数                            |
+---------------------------------+---------------------------------------------+
| suboptions/max                  | 最大订阅选项总数                            |
+---------------------------------+---------------------------------------------+
| subscribers/max                 | 最大订阅者总数                              |
+---------------------------------+---------------------------------------------+
| subscribers/count               | 当前订阅者数量                              |
+---------------------------------+---------------------------------------------+
| subscriptions/max               | 最大订阅数量                                |
+---------------------------------+---------------------------------------------+
| subscriptions/count             | 当前订阅总数                                |
+---------------------------------+---------------------------------------------+
| subscriptions/shared/count      | 当前共享订阅个数                            |
+---------------------------------+---------------------------------------------+
| subscriptions/shared/max        | 当前共享订阅总数                            |
+---------------------------------+---------------------------------------------+

主题统计
::::::::

+---------------------+---------------------------------------------+
| 主题(Topic)         | 说明                                        |
+---------------------+---------------------------------------------+
| topics/count        | 当前 Topic 总数                             |
+---------------------+---------------------------------------------+
| topics/max          | 最大 Topic 数量                             |
+---------------------+---------------------------------------------+

路由统计
::::::::

+---------------------+---------------------------------------------+
| 主题(Topic)         | 说明                                        |
+---------------------+---------------------------------------------+
| routes/count        | 当前 Routes 总数                            |
+---------------------+---------------------------------------------+
| routes/max          | 最大 Routes 数量                            |
+---------------------+---------------------------------------------+

.. note:: ``topics/count`` 和 ``topics/max`` 与 ``routes/count`` 和 ``routes/max`` 数值上是相等的。

收发流量/报文/消息统计
>>>>>>>>>>>>>>>>>>>>>>

系统主题(Topic)前缀: $SYS/brokers/${node}/metrics/

收发流量统计
::::::::::::

+---------------------+---------------------------------------------+
| 主题(Topic)         | 说明                                        |
+---------------------+---------------------------------------------+
| bytes/received      | 累计接收流量                                |
+---------------------+---------------------------------------------+
| bytes/sent          | 累计发送流量                                |
+---------------------+---------------------------------------------+

MQTT报文收发统计
::::::::::::::::

+-----------------------------+---------------------------------------------+
| 主题(Topic)                 | 说明                                        |
+-----------------------------+---------------------------------------------+
| packets/received            | 累计接收 MQTT 报文                          |
+-----------------------------+---------------------------------------------+
| packets/sent                | 累计发送 MQTT 报文                          |
+-----------------------------+---------------------------------------------+
| packets/connect             | 累计接收 MQTT CONNECT 报文                  |
+-----------------------------+---------------------------------------------+
| packets/connack             | 累计发送 MQTT CONNACK 报文                  |
+-----------------------------+---------------------------------------------+
| packets/publish/received    | 累计接收 MQTT PUBLISH 报文                  |
+-----------------------------+---------------------------------------------+
| packets/publish/sent        | 累计发送 MQTT PUBLISH 报文                  |
+-----------------------------+---------------------------------------------+
| packets/puback/received     | 累计接收 MQTT PUBACK 报文                   |
+-----------------------------+---------------------------------------------+
| packets/puback/sent         | 累计发送 MQTT PUBACK 报文                   |
+-----------------------------+---------------------------------------------+
| packets/puback/missed       | 累计丢失 MQTT PUBACK 报文                   |
+-----------------------------+---------------------------------------------+
| packets/pubrec/received     | 累计接收 MQTT PUBREC 报文                   |
+-----------------------------+---------------------------------------------+
| packets/pubrec/sent         | 累计发送 MQTT PUBREC 报文                   |
+-----------------------------+---------------------------------------------+
| packets/pubrec/missed       | 累计丢失 MQTT PUBREC 报文                   |
+-----------------------------+---------------------------------------------+
| packets/pubrel/received     | 累计接收 MQTT PUBREL 报文                   |
+-----------------------------+---------------------------------------------+
| packets/pubrel/sent         | 累计发送 MQTT PUBREL 报文                   |
+-----------------------------+---------------------------------------------+
| packets/pubrel/missed       | 累计丢失 MQTT PUBREL 报文                   |
+-----------------------------+---------------------------------------------+
| packets/pubcomp/received    | 累计接收 MQTT PUBCOMP 报文                  |
+-----------------------------+---------------------------------------------+
| packets/pubcomp/sent        | 累计发送 MQTT PUBCOMP 报文                  |
+-----------------------------+---------------------------------------------+
| packets/pubcomp/missed      | 累计丢失 MQTT PUBCOMP 报文                  |
+-----------------------------+---------------------------------------------+
| packets/subscribe           | 累计接收 MQTT SUBSCRIBE 报文                |
+-----------------------------+---------------------------------------------+
| packets/suback              | 累计发送 MQTT SUBACK 报文                   |
+-----------------------------+---------------------------------------------+
| packets/unsubscribe         | 累计接收 MQTT UNSUBSCRIBE 报文              |
+-----------------------------+---------------------------------------------+
| packets/unsuback            | 累计发送 MQTT UNSUBACK 报文                 |
+-----------------------------+---------------------------------------------+
| packets/pingreq             | 累计接收 MQTT PINGREQ 报文                  |
+-----------------------------+---------------------------------------------+
| packets/pingresp            | 累计发送 MQTT PINGRESP 报文                 |
+-----------------------------+---------------------------------------------+
| packets/disconnect/received | 累计接收 MQTT DISCONNECT 报文               |
+-----------------------------+---------------------------------------------+
| packets/disconnect/sent     | 累计接收 MQTT DISCONNECT 报文               |
+-----------------------------+---------------------------------------------+
| packets/auth                | 累计接收 Auth 报文                          |
+-----------------------------+---------------------------------------------+

MQTT 消息收发统计
:::::::::::::::::

+--------------------------+---------------------------------------------+
| 主题(Topic)              | 说明                                        |
+--------------------------+---------------------------------------------+
| messages/received        | 累计接收消息                                |
+--------------------------+---------------------------------------------+
| messages/sent            | 累计发送消息                                |
+--------------------------+---------------------------------------------+
| messages/expired         | 累计发送消息                                |
+--------------------------+---------------------------------------------+
| messages/retained        | Retained 消息总数                           |
+--------------------------+---------------------------------------------+
| messages/dropped         | 丢弃消息总数                                |
+--------------------------+---------------------------------------------+
| messages/forward         | 节点转发消息总数                            |
+--------------------------+---------------------------------------------+
| messages/qos0/received   | 累计接受 QoS0 消息                          |
+--------------------------+---------------------------------------------+
| messages/qos0/sent       | 累计发送 QoS0 消息                          |
+--------------------------+---------------------------------------------+
| messages/qos1/received   | 累计接受 QoS1 消息                          |
+--------------------------+---------------------------------------------+
| messages/qos1/sent       | 累计发送 QoS1 消息                          |
+--------------------------+---------------------------------------------+
| messages/qos2/received   | 累计接受 QoS2 消息                          |
+--------------------------+---------------------------------------------+
| messages/qos2/sent       | 累计发送 QoS2 消息                          |
+--------------------------+---------------------------------------------+
| messages/qos2/expired    | QoS2 过期消息总数                           |
+--------------------------+---------------------------------------------+
| messages/qos2/dropped    | QoS2 丢弃消息总数                           |
+--------------------------+---------------------------------------------+

.. _sys_alarms:

Alarms - 系统告警
>>>>>>>>>>>>>>>>>

系统主题(Topic)前缀: $SYS/brokers/${node}/alarms/

+-------------+------------------+
| 主题(Topic) | 说明             |
+-------------+------------------+
| alert       | 新产生的告警     |
+-------------+------------------+
| clear       | 被清除的告警     |
+-------------+------------------+

.. _sys_sysmon:

Sysmon - 系统监控
>>>>>>>>>>>>>>>>>

系统主题(Topic)前缀: $SYS/brokers/${node}/sysmon/

+------------------+--------------------+
| 主题(Topic)      | 说明               |
+------------------+--------------------+
| long_gc          | GC 时间过长警告    |
+------------------+--------------------+
| long_schedule    | 调度时间过长警告   |
+------------------+--------------------+
| large_heap       | Heap 内存占用警告  |
+------------------+--------------------+
| busy_port        | Port 忙警告        |
+------------------+--------------------+
| busy_dist_port   | Dist Port 忙警告   |
+------------------+--------------------+

.. _trace:

追踪
----

EMQ X 消息服务器支持追踪来自某个客户端(Client)，或者发布到某个主题(Topic)的全部消息。

追踪来自客户端(Client)的消息:

.. code:: bash

    $ ./bin/emqx_ctl log primary-level debug

    $ ./bin/emqx_ctl trace start client "clientid" "trace_clientid.log" debug

追踪发布到主题(Topic)的消息:

.. code:: bash

    $ ./bin/emqx_ctl log primary-level debug

    $ ./bin/emqx_ctl trace start topic "t/#" "trace_topic.log" debug

查询追踪:

.. code:: bash

    $ ./bin/emqx_ctl trace list

停止追踪:

.. code:: bash

    $ ./bin/emqx_ctl trace stop client "clientid"

    $ ./bin/emqx_ctl trace stop topic "topic"

.. _rule_engine_examples.dashboard.mysql:

创建 MySQL 规则
----------------

0. 搭建 MySQL 数据库，并设置用户名密码为 root/public，以 MacOS X 为例::

    $ brew install mysql

    $ brew services start mysql

    $ mysql -u root -h localhost -p

      ALTER USER 'root'@'localhost' IDENTIFIED BY 'public';

1. 初始化 MySQL 表::

    $ mysql -u root -h localhost -ppublic

  创建 “test” 数据库::

    CREATE DATABASE test;

  创建 “t_mqtt_msg” 表::

    USE test;

    CREATE TABLE `t_mqtt_msg` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
    `msgid` varchar(64) DEFAULT NULL,
    `topic` varchar(255) NOT NULL,
    `qos` tinyint(1) NOT NULL DEFAULT '0',
    `payload` blob,
    `arrived` datetime NOT NULL,
    PRIMARY KEY (`id`),
    INDEX topic_index(`id`, `topic`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8MB4;

  .. image:: ./_static/images/mysql_init_1@2x.png

2. 创建规则:

  打开 `emqx dashboard <http://127.0.0.1:18083/#/rules>`_，选择左侧的 “规则” 选项卡。

  选择触发事件 “消息发布”，然后填写规则 SQL::

    SELECT * FROM "message.pubish"

  .. image:: ./_static/images/rule_sql_1@2x.png

3. 关联动作:

  在 “响应动作” 界面选择 “添加”，然后在 “动作” 下拉框里选择 “保存数据到 MySQL”。

  .. image:: ./_static/images/rule_action_1@2x.png

4. 填写动作参数:

  “保存数据到 MySQL” 动作需要两个参数：

  1). SQL 模板。这个例子里我们向 MySQL 插入一条数据，SQL 模板为::

    insert into t_mqtt_msg(msgid, topic, qos, payload, arrived) values (${id}, ${topic}, ${qos}, ${payload}, FROM_UNIXTIME(${timestamp}/1000))

  .. image:: ./_static/images/rule_action_2@2x.png

  2). 关联资源的 ID。现在资源下拉框为空，可以点击右上角的 “新建资源” 来创建一个 MySQL 资源:

  .. image:: ./_static/images/rule_action_3@2x.png

  选择 “MySQL 资源”。

5. 填写资源配置:

  数据库名填写 “test”，用户名填写 “root”，密码填写 “publish”，备注为 “MySQL resource to 127.0.0.1:3306 db=test”

  .. image:: ./_static/images/rule_resource_1@2x.png

  点击 “新建” 按钮。

6. 返回响应动作界面，点击 “确认”。

  .. image:: ./_static/images/rule_action_4@2x.png

7. 返回规则创建界面，点击 “新建”。

  .. image:: ./_static/images/rule_overview_1@2x.png

  在规则列表里，点击 “查看” 按钮或规则 ID 连接，可以预览刚才创建的规则:

  .. image:: ./_static/images/rule_overview_2@2x.png

8. 规则已经创建完成，现在发一条数据:

    Topic: "t/a"

    QoS: 1

    Payload: "hello"

  然后检查 MySQL 表，新的 record 是否添加成功:

  .. image:: ./_static/images/mysql_result_1@2x.png

.. _rule_engine_examples.dashboard.pgsql:

创建 PostgreSQL 规则
-----------------------

0. 搭建 PostgreSQL 数据库，以 MacOS X 为例::

    $ brew install postgresql

    $ brew services start postgresql

    ## 使用用户名 root 创建名为 'mqtt' 的数据库
    $ createdb -U root mqtt

    $ psql -U root mqtt

      mqtt=> \dn;
      List of schemas
        Name  | Owner
      --------+-------
       public | shawn
      (1 row)

1. 初始化 PgSQL 表:

  $ psql -U root mqtt

  创建 ``t_mqtt_msg`` 表::

    CREATE TABLE t_mqtt_msg (
    id SERIAL primary key,
    msgid character varying(64),
    sender character varying(64),
    topic character varying(255),
    qos integer,
    retain integer,
    payload text,
    arrived timestamp without time zone
    );

2. 创建规则:

  打开 `emqx dashboard <http://127.0.0.1:18083/#/rules>`_，选择左侧的 “规则” 选项卡。

  选择触发事件 “消息发布”，然后填写规则 SQL::

    SELECT
      *
    FROM
      "message.publish"

  .. image:: ./_static/images/pgsql-rulesql-1@2x.png

3. 关联动作:

  在 “响应动作” 界面选择 “添加”，然后在 “动作” 下拉框里选择 “保存数据到 PostgreSQL”。

  .. image:: ./_static/images/pgsql-action-0@2x.png

4. 填写动作参数:

  “保存数据到 PostgreSQL” 动作需要两个参数：

  1). SQL 模板。这个例子里我们向 PostgreSQL 插入一条数据，SQL 模板为::

    insert into t_mqtt_msg(msgid, topic, qos, retain, payload, arrived) values (${id}, ${topic}, ${qos}, ${retain}, ${payload}, to_timestamp(${timestamp}::double precision /1000)) returning id

  插入数据之前，SQL 模板里的 ${key} 占位符会被替换为相应的值。

  .. image:: ./_static/images/pgsql-action-1@2x.png

  2). 关联资源的 ID。现在资源下拉框为空，可以点击右上角的 “新建资源” 来创建一个 PostgreSQL 资源:

  .. image:: ./_static/images/pgsql-resource-0@2x.png

  选择 “PostgreSQL 资源”。

5. 填写资源配置:

  数据库名填写 “mqtt”，用户名填写 “root”，其他配置保持默认值，然后点击 “测试连接” 按钮，确保连接测试成功。

  最后点击 “新建” 按钮。

  .. image:: ./_static/images/pgsql-resource-1@2x.png

6. 返回响应动作界面，点击 “确认”。

  .. image:: ./_static/images/pgsql-action-2@2x.png

7. 返回规则创建界面，点击 “新建”。

  .. image:: ./_static/images/pgsql-rulesql-2@2x.png

8. 规则已经创建完成，现在发一条数据:

    Topic: "t/1"

    QoS: 0

    Retained: false

    Payload: "hello1"

  然后检查 PostgreSQL 表，新的 record 是否添加成功:

  .. image:: ./_static/images/pgsql-result-1@2x.png

  在规则列表里，可以看到刚才创建的规则的命中次数已经增加了 1:

  .. image:: ./_static/images/pgsql-rulelist-1@2x.png

.. _rule_engine_examples.dashboard.cassa:

创建 Cassandra 规则
---------------------

0. 搭建 Cassandra 数据库，并设置用户名密码为 root/public，以 MacOS X 为例::

    $ brew install cassandra

    ## 修改配置，关闭匿名认证
    $  vim /usr/local/etc/cassandra/cassandra.yaml

       authenticator: PasswordAuthenticator
       authorizer: CassandraAuthorizer

    $ brew services start cassandra

    ## 创建 root 用户
    $ cqlsh -ucassandra -pcassandra

      create user root with password 'public' superuser;

1. 初始化 Cassandra 表::

    $ cqlsh -uroot -ppublic

  创建 "test" 表空间::

    CREATE KEYSPACE test WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '1'}  AND durable_writes = true;

  创建 “t_mqtt_msg” 表::

    USE test;

    CREATE TABLE t_mqtt_msg (
      msgid text,
      topic text,
      qos int,
      payload text,
      retain int,
      arrived timestamp,
      PRIMARY KEY (msgid, topic)
    );

2. 创建规则:

  打开 `emqx dashboard <http://127.0.0.1:18083/#/rules>`_，选择左侧的 “规则” 选项卡。

  选择触发事件 “消息发布”，然后填写规则 SQL::

    SELECT
      *, flags.retain as retain
    FROM
      "message.publish"

  .. image:: ./_static/images/pgsql-rulesql-1@2x.png

3. 关联动作:

  在 “响应动作” 界面选择 “添加”，然后在 “动作” 下拉框里选择 “保存数据到 Cassandra”。

  .. image:: ./_static/images/cass-action-0@2x.png

4. 填写动作参数:

  “保存数据到 Cassandra” 动作需要两个参数：

  1). SQL 模板。这个例子里我们向 Cassandra 插入一条数据，SQL 模板为::

    insert into t_mqtt_msg(msgid, topic, qos, payload, retain, arrived) values (${id}, ${topic}, ${qos}, ${payload}, ${retain}, ${timestamp})

  插入数据之前，SQL 模板里的 ${key} 占位符会被替换为相应的值。

  2). 关联资源的 ID。初始状况下，资源下拉框为空，现点击右上角的 “新建资源” 来创建一个 Cassandra 资源。

5. 填写资源配置:

  Keysapce 填写 “test”，用户名填写 “root”，密码填写 “public” 其他配置保持默认值，然后点击 “测试连接” 按钮，确保连接测试成功。

  .. image:: ./_static/images/cass-resoure-1.png

  点击 “新建” 按钮，完成资源的创建。

6. 自动返回响应动作界面，点击 “确认” 完成响应动作的创建；自动返回规则创建页面，在点击 “新建” 完成规则创建

  .. image:: ./_static/images/cass-rule-overview.png

7. 现在发送一条数据，测试该规则::

    Topic: "t/cass"
    QoS: 1
    Retained: true
    Payload: "hello"

  然后检查 Cassandra 表，可以看到该消息已成功保存:

  .. image:: ./_static/images/cass-rule-result@2x.png


.. _rule_engine_examples.dashboard.mongo:

创建 MongoDB 规则
------------------

0. 搭建 MongoDB 数据库，并设置用户名密码为 root/public，以 MacOS X 为例::

    $ brew install mongodb
    $ brew services start mongodb

    ## 新增 root/public 用户
    $ use mqtt;
    $ db.createUser({user: "root", pwd: "public", roles: [{role: "readWrite", db: "mqtt"}]});

    ## 修改配置，关闭匿名认证
    $ vim /usr/local/etc/mongod.conf

      security:
        authorization: enabled

    $ brew services restart mongodb

1. 初始化 MongoDB 表::

    $ mongo 127.0.0.1/mqtt -uroot -ppublic

      db.createCollection("t_mqtt_msg");

2. 创建规则:

  打开 `emqx dashboard <http://127.0.0.1:18083/#/rules>`_，选择左侧的 “规则” 选项卡。

  选择触发事件 “消息发布”，然后填写规则 SQL::

    SELECT
      *, flags.retain as retain
    FROM
      "message.publish"

  .. image:: ./_static/images/pgsql-rulesql-1@2x.png

3. 关联动作:

  在 “响应动作” 界面选择 “添加”，然后在 “动作” 下拉框里选择 “保存数据到 MongoDB”。

  .. image:: ./_static/images/mongo-action-0@2x.png

4. 填写动作参数:

  “保存数据到 MongoDB” 动作需要三个参数：

  1). Collection 名称。这个例子我们向刚刚新建的 collection 插入数据，填 “t_mqtt_msg”

  2). Selector 模板。这个例子里我们向 MongoDB 插入一条数据，Selector 模板为::

    msgid=${id},topic=${topic},qos=${qos},payload=${payload},retain=${retain},arrived=${timestamp}

  插入数据之前，Selector 模板里的 ${key} 占位符会被替换为相应的值。

  3). 关联资源的 ID。初始状况下，资源下拉框为空，现点击右上角的 “新建资源” 来创建一个 MongoDB 单节点 资源。

5. 填写资源配置:

  数据库名称 填写 “mqtt”，用户名填写 “root”，密码填写 “public”，连接认证源填写 “mqtt” 其他配置保持默认值，然后点击 “测试连接” 按钮，确保连接测试成功。

  .. image:: ./_static/images/mongo-resoure-1.png

  点击 “新建” 按钮，完成资源的创建。

6. 自动返回响应动作界面，点击 “确认” 完成响应动作的创建；自动返回规则创建页面，在点击 “新建” 完成规则创建

  .. image:: ./_static/images/mongo-rule-overview.png

7. 现在发送一条数据，测试该规则::

    Topic: "t/mongo"
    QoS: 1
    Retained: true
    Payload: "hello"

  然后检查 MongoDB 表，可以看到该消息已成功保存:

  .. image:: ./_static/images/mongo-rule-result@2x.png


.. _rule_engine_examples.dashboard.dynamodb:

创建 DynamoDB 规则
--------------------

0. 搭建 DynamoDB 数据库，以 MacOS X 为例::

    $ brew install dynamodb-local

    $ dynamodb-local

1. 创建 DynamoDB 表定义文件 mqtt_msg.json :

.. code-block:: json

     {
         "TableName": "mqtt_msg",
         "KeySchema": [
             { "AttributeName": "msgid", "KeyType": "HASH" }
         ],
         "AttributeDefinitions": [
             { "AttributeName": "msgid", "AttributeType": "S" }
         ],
         "ProvisionedThroughput": {
             "ReadCapacityUnits": 5,
             "WriteCapacityUnits": 5
         }
     }

2. 初始化 DynamoDB 表::

    $ aws dynamodb create-table --cli-input-json file://mqtt_msg.json --endpoint-url http://localhost:8000

3. 创建规则:

  打开 `emqx dashboard <http://127.0.0.1:18083/#/rules>`_，选择左侧的 “规则” 选项卡。

  选择触发事件 “消息发布”，然后填写规则 SQL::

    SELECT
     msgid as id, topic, payload
    FROM
      "message.pubish"

  .. image:: ./_static/images/dynamo-rulesql-0.png

4. 关联动作:

  在 “响应动作” 界面选择 “添加”，然后在 “动作” 下拉框里选择 “保存数据到 DynamoDB”。

  .. image:: ./_static/images/dynamo-action-0.png

5. 填写动作参数:

  “保存数据到 DynamoDB” 动作需要两个参数：

  1). DynamoDB 表名。这个例子里我们设置的表名为 "mqtt_msg"

  2). DynamoDB Hash Key。这个例子里我们设置的 Hash Key 要与表定义的一致

  3). DynamoDB Range Key。由于我们表定义里没有设置 Range Key。这个例子里我们把 Range Key 设置为空。

  .. image:: ./_static/images/dynamo-action-1.png

  4). 关联资源的 ID。现在资源下拉框为空，可以点击右上角的 “新建资源” 来创建一个 DynamoDB 资源:

  .. image:: ./_static/images/dynamo-resource-0.png

  选择 “DynamoDB 资源”。

6. 填写资源配置:

  区域名填写“us-west-2”

  服务器地址填写“http://localhost:8000”

  连接访问ID填写“AKIAU5IM2XOC7AQWG7HK”

  连接访问密钥填写“TZt7XoRi+vtCJYQ9YsAinh19jR1rngm/hxZMWR2P”

  .. image:: ./_static/images/dynamo-resource-1.png

  点击 “新建” 按钮。

7. 返回响应动作界面，点击 “确认”。

  .. image:: ./_static/images/dynamo-action-2.png

8. 返回规则创建界面，点击 “新建”。

  .. image:: ./_static/images/dynamo-rulesql-1.png

9. 规则已经创建完成，现在发一条数据:

    Topic: "t/a"

    QoS: 1

    Payload: "hello"

  然后检查 DynamoDB 的 mqtt_msg 表，新的 record 是否添加成功:

  .. image:: ./_static/images/dynamo-result-0.png

  在规则列表里，可以看到刚才创建的规则的命中次数已经增加了 1:

  .. image:: ./_static/images/dynamo-result-1.png

.. _rule_engine_examples.dashboard.redis:

创建 Redis 规则
-----------------

0. 搭建 Redis 环境，以 MaxOS X 为例::

    $ wget http://download.redis.io/releases/redis-4.0.14.tar.gz
    $ tar xzf redis-4.0.14.tar.gz
    $ cd redis-4.0.14
    $ make && make install

    启动 redis
    $ redis-server


1. 创建规则:

  打开 `emqx dashboard <http://127.0.0.1:18083/#/rules>`_，选择左侧的 “规则” 选项卡。

  选择触发事件 “消息发布”，然后填写规则 SQL::

    SELECT
      *
    FROM
      "message.publish"
    WHERE
      topic =~ 't/#'

  .. image:: ./_static/images/redis-rulesql-0@2x.png

2. 关联动作:

  在 “响应动作” 界面选择 “添加”，然后在 “动作” 下拉框里选择 “保存数据到 Redis”。

  .. image:: ./_static/images/redis-action-0@2x.png

3. 填写动作参数:

  “保存数据到 Redis 动作需要两个参数：

  1). Redis 的命令::

    HMSET mqtt:msg:${id} id ${id} from ${client_id} qos ${qos} topic ${topic} payload ${payload} retain ${retain} ts ${timestamp}

  2). 关联资源。现在资源下拉框为空，可以点击右上角的 “新建资源” 来创建一个 Redis 资源:

  .. image:: ./_static/images/redis-resource-0@2x.png

  选择 Redis 单节点模式资源”。

  .. image:: ./_static/images/redis-resource-1@2x.png

4. 填写资源配置:

   填写真实的 Redis 服务器地址，其他配置保持默认值，然后点击 “测试连接” 按钮，确保连接测试成功。

  最后点击 “新建” 按钮。

  .. image:: ./_static/images/redis-resource-2@2x.png

5. 返回响应动作界面，点击 “确认”。

  .. image:: ./_static/images/redis-action-1@2x.png

6. 返回规则创建界面，点击 “新建”。

  .. image:: ./_static/images/redis-rulesql-1@2x.png

7. 规则已经创建完成，现在发一条数据:

    Topic: "t/1"

    QoS: 0

    Retained: false

    Payload: "hello"

  然后通过 Redis 命令去查看消息是否生产成功::

  $ redis-cli

  KEYS mqtt:msg*

  hgetall Key

  .. image:: ./_static/images/redis-cli.png

  在规则列表里，可以看到刚才创建的规则的命中次数已经增加了 1:

  .. image:: ./_static/images/redis-rulelist-0@2x.png


.. _rule_engine_examples.dashboard.opentsdb:

创建 OpenTSDB 规则
--------------------

0. 搭建 OpenTSDB 数据库环境，以 MaxOS X 为例::

    $ docker pull petergrace/opentsdb-docker

    $ docker run -d --name opentsdb -p 4242:4242 petergrace/opentsdb-docker

1. 创建规则:

  打开 `emqx dashboard <http://127.0.0.1:18083/#/rules>`_，选择左侧的 “规则” 选项卡。

  选择触发事件 “消息发布”，然后填写规则 SQL::

    SELECT
      payload.metric as metric, payload.tags as tags, payload.value as value
    FROM
      "message.publish"

  .. image:: ./_static/images/opentsdb-rulesql-0@2x.png

2. 关联动作:

  在 “响应动作” 界面选择 “添加”，然后在 “动作” 下拉框里选择 “保存数据到 OpenTSDB”。

  .. image:: ./_static/images/opentsdb-action-0@2x.png

3. 填写动作参数:

  “保存数据到 OpenTSDB” 动作需要六个参数:

  1). 详细信息。是否需要 OpenTSDB Server 返回存储失败的 data point 及其原因的列表，默认为 false。

  2). 摘要信息。是否需要 OpenTSDB Server 返回 data point 存储成功与失败的数量，默认为 true。

  3). 最大批处理数量。消息请求频繁时允许 OpenTSDB 驱动将多少个 Data Points 合并为一次请求，默认为 20。

  4). 是否同步调用。指定 OpenTSDB Server 是否等待所有数据都被写入后才返回结果，默认为 false。

  5). 同步调用超时时间。同步调用最大等待时间，默认为 0。

  6). 关联资源。现在资源下拉框为空，可以点击右上角的 “新建资源” 来创建一个 OpenTSDB 资源:

  .. image:: ./_static/images/opentsdb-action-1@2x.png

  选择 “OpenTSDB 资源”:

  .. image:: ./_static/images/opentsdb-resource-0@2x.png

4. 填写资源配置:

  本示例中所有配置保持默认值即可，点击 “测试连接” 按钮，确保连接测试成功。

  最后点击 “新建” 按钮。

  .. image:: ./_static/images/opentsdb-resource-1@2x.png

5. 返回响应动作界面，点击 “确认”。

  .. image:: ./_static/images/opentsdb-action-2@2x.png

6. 返回规则创建界面，点击 “新建”。

  .. image:: ./_static/images/opentsdb-rulesql-1@2x.png

7. 规则已经创建完成，现在发一条消息:

    Topic: "t/1"

    QoS: 0

    Retained: false

    Payload: "{\"metric\":\"cpu\",\"tags\":{\"host\":\"serverA\"},\"value\":12}"

  我们通过 Postman 或者 curl 命令，向 OpenTSDB Server 发送以下请求::

    POST /api/query HTTP/1.1
    Host: 127.0.0.1:4242
    Content-Type: application/json
    cache-control: no-cache
    Postman-Token: 69af0565-27f8-41e5-b0cd-d7c7f5b7a037
    {
        "start": 1560409825000,
        "queries": [
            {
                "aggregator": "last",
                "metric": "cpu",
                "tags": {
                    "host": "*"
                }
            }
        ],
        "showTSUIDs": "true",
        "showQuery": "true",
        "delete": "false"
    }
    ------WebKitFormBoundary7MA4YWxkTrZu0gW--

  如果 data point 存储成功，将会得到以下应答:

  .. code-block:: json

    [
      {
          "metric": "cpu",
          "tags": {
              "host": "serverA"
          },
          "aggregateTags": [],
          "query": {
              "aggregator": "last",
              "metric": "cpu",
              "tsuids": null,
              "downsample": null,
              "rate": false,
              "filters": [
                  {
                      "tagk": "host",
                      "filter": "*",
                      "group_by": true,
                      "type": "wildcard"
                  }
              ],
              "index": 0,
              "tags": {
                  "host": "wildcard(*)"
              },
              "rateOptions": null,
              "filterTagKs": [
                  "AAAC"
              ],
              "explicitTags": false
          },
          "tsuids": [
              "000002000002000007"
          ],
          "dps": {
              "1561532453": 12
          }
      }
    ]

  在规则列表里，可以看到刚才创建的规则的命中次数已经增加了 1:

  .. image:: ./_static/images/opentsdb-rulelist-1@2x.png

.. _rule_engine_examples.dashboard.timescaledb:

创建 TimescaleDB 规则
----------------------

0. 搭建 TimescaleDB 数据库环境，以 MaxOS X 为例::

    $ docker pull timescale/timescaledb

    $ docker run -d --name timescaledb -p 5432:5432 -e POSTGRES_PASSWORD=password timescale/timescaledb:latest-pg11

    $ docker exec -it timescaledb psql -U postgres

    ## 创建并连接 tutorial 数据库
    > CREATE database tutorial;

    > \c tutorial

    > CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;

1. 初始化 TimescaleDB 表::

    $ docker exec -it timescaledb psql -U postgres -d tutorial

  创建 ``conditions`` 表::

    CREATE TABLE conditions (
      time        TIMESTAMPTZ       NOT NULL,
      location    TEXT              NOT NULL,
      temperature DOUBLE PRECISION  NULL,
      humidity    DOUBLE PRECISION  NULL
    );

    SELECT create_hypertable('conditions', 'time');

2. 创建规则:

  打开 `emqx dashboard <http://127.0.0.1:18083/#/rules>`_，选择左侧的 “规则” 选项卡。

  选择触发事件 “消息发布”，然后填写规则 SQL::

    SELECT
      payload.temp as temp,
      payload.humidity as humidity,
      payload.location as location
    FROM
      "message.publish"

  .. image:: ./_static/images/timescaledb-rulesql-0@2x.png

3. 关联动作:

  在 “响应动作” 界面选择 “添加”，然后在 “动作” 下拉框里选择 “保存数据到 TimescaleDB”。

  .. image:: ./_static/images/timescaledb-action-0@2x.png

4. 填写动作参数:

  “保存数据到 TimescaleDB” 动作需要两个参数：

  1). SQL 模板。这个例子里我们向 TimescaleDB 插入一条数据，SQL 模板为::

    insert into conditions(time, location, temperature, humidity) values (NOW(), ${location}, ${temp}, ${humidity})

  插入数据之前，SQL 模板里的 ${key} 占位符会被替换为相应的值。

  2). 关联资源。现在资源下拉框为空，可以点击右上角的 “新建资源” 来创建一个 TimescaleDB 资源:

  .. image:: ./_static/images/timescaledb-resource-0@2x.png

  选择 “TimescaleDB 资源”。

  .. image:: ./_static/images/timescaledb-resource-1@2x.png

5. 填写资源配置:

  数据库名填写 “tutorial”，用户名填写 “postgres”，密码填写 “password”，其他配置保持默认值，然后点击 “测试连接” 按钮，确保连接测试成功。

  最后点击 “新建” 按钮。

  .. image:: ./_static/images/timescaledb-resource-2@2x.png

6. 返回响应动作界面，点击 “确认”。

  .. image:: ./_static/images/timescaledb-action-1@2x.png

7. 返回规则创建界面，点击 “新建”。

  .. image:: ./_static/images/timescaledb-rulesql-1@2x.png

8. 规则已经创建完成，现在发一条数据:

    Topic: "t/1"

    QoS: 0

    Retained: false

    Payload: "{\"temp\":24,\"humidity\":30,\"location\":\"hangzhou\"}"

  然后检查 TimescaleDB 表，新的 record 是否添加成功::

    tutorial=# SELECT * FROM conditions LIMIT 100;
                time              | location | temperature | humidity
    -------------------------------+----------+-------------+----------
    2019-06-27 01:41:08.752103+00 | hangzhou |          24 |       30

  在规则列表里，可以看到刚才创建的规则的命中次数已经增加了 1:

  .. image:: ./_static/images/timescaledb-rulelist-0@2x.png

.. _rule_engine_examples.dashboard.influxdb:

创建 InfluxDB 规则
--------------------

0. 搭建 InfluxDB 数据库环境，以 MacOS X 为例::

    $ docker pull influxdb

    $ git clone -b v1.0.0 https://github.com/palkan/influx_udp.git

    $ cd influx_udp

    $ docker run --name=influxdb --rm -d -p 8086:8086 -p 8089:8089/udp -v ${PWD}/files/influxdb.conf:/etc/influxdb/influxdb.conf:ro -e INFLUXDB_DB=db influxdb:latest

1. 创建规则:

  打开 `emqx dashboard <http://127.0.0.1:18083/#/rules>`_，选择左侧的 “规则” 选项卡。

  选择触发事件 “消息发布”，然后填写规则 SQL::

    SELECT
      payload.host as host,
      payload.location as location,
      payload.internal as internal,
      payload.external as external
    FROM
      "message.publish"

  .. image:: ./_static/images/influxdb-rulesql-0@2x.png

2. 关联动作:

  在 “响应动作” 界面选择 “添加”，然后在 “动作” 下拉框里选择 “保存数据到 InfluxDB”。

  .. image:: ./_static/images/influxdb-action-0@2x.png

3. 填写动作参数:

  “保存数据到 InfluxDB” 动作需要六个参数：

  1). Measurement。指定写入到 InfluxDB 的 data point 的 measurement。

  2). Field Keys。指定写入到 InfluxDB 的 data point 的 fields 的值从哪里获取。

  3). Tags Keys。指定写入到 InfluxDB 的 data point 的 tags 的值从哪里获取。

  4). Timestamp Key。指定写入到 InfluxDB 的 data point 的 timestamp 的值从哪里获取。

  5). 设置时间戳。未指定 Timestamp Key 时是否自动生成。

  6). 关联资源。现在资源下拉框为空，可以点击右上角的 “新建资源” 来创建一个 InfluxDB 资源:

  .. image:: ./_static/images/influxdb-action-1@2x.png

  选择 “InfluxDB 资源”:

  .. image:: ./_static/images/influxdb-resource-0@2x.png

4. 填写资源配置:

  本示例中所有配置保持默认值即可，点击 “测试连接” 按钮，确保连接测试成功。

  最后点击 “新建” 按钮。

  .. image:: ./_static/images/influxdb-resource-1@2x.png

5. 返回响应动作界面，点击 “确认”。

6. 返回规则创建界面，点击 “新建”。

  .. image:: ./_static/images/influxdb-rulesql-1@2x.png

7. 规则已经创建完成，现在发一条消息:

    Topic: "t/1"

    QoS: 0

    Retained: false

    Payload: "{\"host\":\"serverA\",\"location\":\"roomA\",\"internal\":25,\"external\":37}"

  然后检查 InfluxDB，新的 data point 是否添加成功::

    $ docker exec -it influxdb influx

    > use db
    Using database db
    > select * from "temperature"
    name: temperature
    time                external host    internal location
    ----                -------- ----    -------- --------
    1561535778444457348 35       serverA 25       roomA

  在规则列表里，可以看到刚才创建的规则的命中次数已经增加了 1:

  .. image:: ./_static/images/influxdb-rulelist-0@2x.png

.. _rule_engine_examples.dashboard.webhook:

创建 WebHook 规则
-------------------

0. 搭建 Web 服务，这里使用 ``nc`` 命令做一个简单的Web 服务::

    $ while true; do echo -e "HTTP/1.1 200 OK\n\n $(date)" | nc -l 127.0.0.1 9901; done;

1. 创建规则:

  打开 `emqx dashboard <http://127.0.0.1:18083/#/rules>`_，选择左侧的 “规则” 选项卡。

  选择触发事件 “消息发布”，然后填写规则 SQL::

    SELECT
      *
    FROM
      "message.publish"

  .. image:: ./_static/images/webhook-rulesql-1.png

2. 关联动作:

  在 “响应动作” 界面选择 “添加”，然后在 “动作” 下拉框里选择 “发送数据到 Web 服务”。

  .. image:: ./_static/images/webhook-action-1.png

3. 给动作关联资源:

  现在资源下拉框为空，可以点击右上角的 “新建资源” 来创建一个 WebHook 资源:

  .. image:: ./_static/images/webhook-action-2.png

  选择 “WebHook 资源”:

  .. image:: ./_static/images/webhook-resource-1.png

4. 填写资源配置:

  填写 “请求 URL” 和请求头(可选)::

    http://127.0.0.1:9901

  点击 “测试连接” 按钮，确保连接测试成功，最后点击 “新建” 按钮:

  .. image:: ./_static/images/webhook-resource-2.png

5. 返回响应动作界面，点击 “确认”。

  .. image:: ./_static/images/webhook-action-3.png

6. 返回规则创建界面，点击 “新建”。

  .. image:: ./_static/images/webhook-rule-create.png

  规则已经创建完成，规则列表里展示出了新创建的规则:

  .. image:: ./_static/images/webhook-rulelist-1.png

7. 发一条消息::

    Topic: "t/1"

    QoS: 1

    Payload: "Hello web server"

  然后检查 Web 服务是否收到消息:

  .. image:: ./_static/images/webhook-result-1.png

.. _rule_engine_examples.dashboard.kafka:

创建 Kafka 规则
-----------------

0. 搭建 Kafka 环境，以 MaxOS X 为例::

    $ wget http://apache.claz.org/kafka/2.3.0/kafka_2.12-2.3.0.tgz

    $ tar -xzf  kafka_2.12-2.3.0.tgz

    $ cd kafka_2.12-2.3.0

    启动 Zookeeper
    $ ./bin/zookeeper-server-start.sh config/zookeeper.properties
    启动 Kafka
    $ ./bin/kafka-server-start.sh config/server.properties


1. 创建 Kafka 的主题::

    $ ./bin/kafka-topics.sh --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic testTopic --create

    .. note:: 创建 Kafka Rule 之前必须先在 Kafka 中创建好主题，否则创建 Kafka Rule 失败。

2. 创建规则:

  打开 `emqx dashboard <http://127.0.0.1:18083/#/rules>`_，选择左侧的 “规则” 选项卡。

  选择触发事件 “消息发布”，然后填写规则 SQL::

    SELECT
      *
    FROM
      "message.publish"
    WHERE
      topic =~ 't/#'

  .. image:: ./_static/images/kafka-rulesql-0@2x.png

3. 关联动作:

  在 “响应动作” 界面选择 “添加”，然后在 “动作” 下拉框里选择 “桥接数据到 Kafka”。

  .. image:: ./_static/images/kafka-action-0@2x.png

4. 填写动作参数:

  “保存数据到 Kafka 动作需要两个参数：

  1). Kafka 的消息主题

  2). 关联资源。现在资源下拉框为空，可以点击右上角的 “新建资源” 来创建一个 Kafka 资源:

  .. image:: ./_static/images/kafka-resource-0@2x.png

  选择 Kafka 资源”。

  .. image:: ./_static/images/kafka-resource-1@2x.png

5. 填写资源配置:

   填写真实的 Kafka 服务器地址，多个地址用,分隔，其他配置保持默认值，然后点击 “测试连接” 按钮，确保连接测试成功。

  最后点击 “新建” 按钮。

  .. image:: ./_static/images/kafka-resource-2@2x.png

6. 返回响应动作界面，点击 “确认”。

  .. image:: ./_static/images/kafka-action-1@2x.png

7. 返回规则创建界面，点击 “新建”。

  .. image:: ./_static/images/kafka-rulesql-1@2x.png

8. 规则已经创建完成，现在发一条数据:

    Topic: "t/1"

    QoS: 0

    Retained: false

    Payload: "hello"

  然后通过 Kafka 命令去查看消息是否生产成功::

  $ ./bin/kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092  --topic testTopic --from-beginning

    .. image:: ./_static/images/kafka-consumer.png

  在规则列表里，可以看到刚才创建的规则的命中次数已经增加了 1:

  .. image:: ./_static/images/kafka-rulelist-0@2x.png

.. _rule_engine_examples.dashboard.pulsar:

创建 Pulsar 规则
------------------

0. 搭建 Pulsar 环境，以 MaxOS X 为例::

    $ wget http://apache.mirrors.hoobly.com/pulsar/pulsar-2.3.2/apache-pulsar-2.3.2-bin.tar.gz

    $ tar xvfz apache-pulsar-2.3.2-bin.tar.gz

    $ cd apache-pulsar-2.3.2

    启动 Pulsar
    $ ./bin/pulsar standalone

1. 创建 Pulsar 的主题::

    $ ./bin/pulsar-admin topics create-partitioned-topic -p 5 testTopic

2. 创建规则:

  打开 `emqx dashboard <http://127.0.0.1:18083/#/rules>`_，选择左侧的 “规则” 选项卡。

  选择触发事件 “消息发布”，然后填写规则 SQL::

    SELECT
      *
    FROM
      "message.publish"
    WHERE
      topic =~ 't/#'

  .. image:: ./_static/images/pulsar-rulesql-0@2x.png

3. 关联动作:

  在 “响应动作” 界面选择 “添加”，然后在 “动作” 下拉框里选择 “桥接数据到 Pulsar”。

  .. image:: ./_static/images/pulsar-action-0@2x.png

4. 填写动作参数:

  “保存数据到 Pulsar 动作需要两个参数：

  1). Pulsar 的消息主题

  2). 关联资源。现在资源下拉框为空，可以点击右上角的 “新建资源” 来创建一个 Pulsar 资源:

  .. image:: ./_static/images/pulsar-resource-0@2x.png

  选择 Pulsar 资源”。

  .. image:: ./_static/images/pulsar-resource-1@2x.png

5. 填写资源配置:

   填写真实的 Pulsar 服务器地址，多个地址用,分隔，其他配置保持默认值，然后点击 “测试连接” 按钮，确保连接测试成功。

  最后点击 “新建” 按钮。

  .. image:: ./_static/images/pulsar-resource-2@2x.png

6. 返回响应动作界面，点击 “确认”。

  .. image:: ./_static/images/pulsar-action-1@2x.png

7. 返回规则创建界面，点击 “新建”。

  .. image:: ./_static/images/pulsar-rulesql-1@2x.png

8. 规则已经创建完成，现在发一条数据:

    Topic: "t/1"

    QoS: 0

    Retained: false

    Payload: "hello"

  然后通过 Pulsar 命令去查看消息是否生产成功::

  $ ./bin/pulsar-client consume testTopic  -s "sub-name" -n 1000

    .. image:: ./_static/images/pulsar-consumer.png

  在规则列表里，可以看到刚才创建的规则的命中次数已经增加了 1:

  .. image:: ./_static/images/pulsar-rulelist-0@2x.png

.. _rule_engine_examples.dashboard.rabbit:

创建 RabbitMQ 规则
--------------------

0. 搭建 RabbitMQ 环境，以 MaxOS X 为例::

    $ brew install rabbitmq

    启动 rabbitmq
    $ rabbitmq-server

1. 创建规则:

  打开 `emqx dashboard <http://127.0.0.1:18083/#/rules>`_，选择左侧的 “规则” 选项卡。

  选择触发事件 “消息发布”，然后填写规则 SQL::

    SELECT
      *
    FROM
      "message.publish"
    WHERE
      topic =~ 't/#'

  .. image:: ./_static/images/rabbit-rulesql-0.png

2. 关联动作:

  在 “响应动作” 界面选择 “添加”，然后在 “动作” 下拉框里选择 “桥接数据到 RabbitMQ”。

  .. image:: ./_static/images/rabbit-action-0.png

3. 填写动作参数:

  “桥接数据到 RabbitMQ 动作需要四个参数：

  1). RabbitMQ Exchange。这个例子里我们设置 Exchange 为 "messages"，

  2). RabbitMQ Exchange Type。这个例子我们设置 Exchange Type 为 "topic"

  3). RabbitMQ Routing Key。这个例子我们设置 Routing Key 为 "test"

  4). 关联资源。现在资源下拉框为空，可以点击右上角的 “新建资源” 来创建一个 RabbitMQ 资源:

  .. image:: ./_static/images/rabbit-action-1.png

  选择 RabbitMQ 资源。

  .. image:: ./_static/images/rabbit-resource-0.png

4. 填写资源配置:

   填写真实的 RabbitMQ 服务器地址，其他配置保持默认值，然后点击 “测试连接” 按钮，确保连接测试成功。

  最后点击 “新建” 按钮。

  .. image:: ./_static/images/rabbit-resource-1.png

5. 返回响应动作界面，点击 “确认”。

  .. image:: ./_static/images/rabbit-action-2.png

6. 返回规则创建界面，点击 “新建”。

  .. image:: ./_static/images/rabbit-rulesql-1.png

7. 规则已经创建完成，现在发一条数据:

    Topic: "t/1"

    QoS: 0

    Retained: false

    Payload: "Hello, World!"

  然后通过 amqp 协议的客户端查看消息是否发布成功

  .. image:: ./_static/images/rabbit-subscriber-0.png

  在规则列表里，可以看到刚才创建的规则的命中次数已经增加了 1:

  .. image:: ./_static/images/rabbit-rulelist-0.png

.. _rule_engine_examples.dashboard.bridge_mqtt:

创建 BridgeMQTT 规则
----------------------

0. 搭建 MQTT Broker 环境，以 MaxOS X 为例::

    $ brew install mosquitto

    启动 mosquitto
    $ mosquitto

1. 创建规则:

  打开 `emqx dashboard <http://127.0.0.1:18083/#/rules>`_，选择左侧的 “规则” 选项卡。

  选择触发事件 “消息发布”，然后填写规则 SQL::

    SELECT
      *
    FROM
      "message.publish"
    WHERE
      topic =~ 't/#'

  .. image:: ./_static/images/mqtt-rulesql-0.png

2. 关联动作:

  在 “响应动作” 界面选择 “添加”，然后在 “动作” 下拉框里选择 “桥接数据到 MQTT Broker”。

  .. image:: ./_static/images/mqtt-action-0.png

3. 填写动作参数:

  "桥接数据到 MQTT Broker" 动作只需要一个参数：

  关联资源。现在资源下拉框为空，可以点击右上角的 “新建资源” 来创建一个 MQTT Bridge 资源:

  .. image:: ./_static/images/mqtt-action-1.png

  选择 MQTT Bridge 资源。

  .. image:: ./_static/images/mqtt-resource-0.png

4. 填写资源配置:

   填写真实的 mosquitto 服务器地址，其他配置保持默认值，然后点击 “测试连接” 按钮，确保连接测试成功。

  最后点击 “新建” 按钮。

  .. image:: ./_static/images/mqtt-resource-1.png

6. 返回响应动作界面，点击 “确认”。

  .. image:: ./_static/images/mqtt-action-2.png

7. 返回规则创建界面，点击 “新建”。

  .. image:: ./_static/images/mqtt-rulesql-1.png

8. 规则已经创建完成，现在发一条数据:

    Topic: "t/1"

    QoS: 0

    Retained: false

    Payload: "Hello, World!"

  然后通过 mqtt 客户端查看消息是否发布成功

  .. image:: ./_static/images/mqtt-result-0.png

  在规则列表里，可以看到刚才创建的规则的命中次数已经增加了 1:

  .. image:: ./_static/images/mqtt-rulelist-0.png

.. _rule_engine_examples.dashboard.bridge_rpc:

创建 BridgeRPC 规则
---------------------

0. 搭建 EMQX Broker 环境，以 MaxOS X 为例::

    $ brew tap emqx/emqx/emqx

    $ brew install emqx

    启动 emqx
    $ emqx console

1. 创建规则:

  打开 `emqx dashboard <http://127.0.0.1:18083/#/rules>`_，选择左侧的 “规则” 选项卡。

  选择触发事件 “消息发布”，然后填写规则 SQL::

    SELECT
      *
    FROM
      "message.publish"
    WHERE
      topic =~ 't/#'

  .. image:: ./_static/images/rpc-rulesql-0.png

2. 关联动作:

  在 “响应动作” 界面选择 “添加”，然后在 “动作” 下拉框里选择 “桥接数据到 MQTT Broker”。

  .. image:: ./_static/images/rpc-action-0.png

3. 填写动作参数:

  桥接数据到 MQTT Broker 动作只需要一个参数：

  关联资源。现在资源下拉框为空，可以点击右上角的 “新建资源” 来创建一个 RPC Bridge 资源:

  .. image:: ./_static/images/rpc-action-1.png

  选择 RPC Bridge 资源。

  .. image:: ./_static/images/rpc-resource-0.png

4. 填写资源配置:

   填写真实的 emqx 节点名，其他配置保持默认值，然后点击 “测试连接” 按钮，确保连接测试成功。

  最后点击 “新建” 按钮。

  .. image:: ./_static/images/rpc-resource-1.png

6. 返回响应动作界面，点击 “确认”。

  .. image:: ./_static/images/rpc-action-2.png

7. 返回规则创建界面，点击 “新建”。

  .. image:: ./_static/images/rpc-rulesql-1.png

8. 规则已经创建完成，现在发一条数据:

    Topic: "t/1"

    QoS: 0

    Retained: false

    Payload: "Hello, World!"

  然后通过 mqtt 客户端查看消息是否发布成功

  .. image:: ./_static/images/rpc-result-0.png

  在规则列表里，可以看到刚才创建的规则的命中次数已经增加了 1:

  .. image:: ./_static/images/rpc-rulelist-0.png



.. _emqx_auth_clientid: https://github.com/emqx/emqx-auth-clientid
.. _emqx_auth_username: https://github.com/emqx/emqx-auth-username
.. _emqx_auth_ldap:     https://github.com/emqx/emqx-auth-ldap
.. _emqx_auth_http:     https://github.com/emqx/emqx-auth-http
.. _emqx_auth_mysql:    https://github.com/emqx/emqx-auth-mysql
.. _emqx_auth_pgsql:    https://github.com/emqx/emqx-auth-pgsql
.. _emqx_auth_redis:    https://github.com/emqx/emqx-auth-redis
.. _emqx_auth_mongo:    https://github.com/emqx/emqx-auth-mongo
.. _emqx_auth_jwt:      https://github.com/emqx/emqx-auth-jwt
.. _emqx_psk_file:      https://github.com/emqx/emqx-psk-file
