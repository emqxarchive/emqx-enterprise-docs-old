
.. _cli:

========
管理命令
========

EMQ X企业版提供了'./bin/emqx_ctl'的管理命令行。

----------
status命令
----------

查询EMQ X服务器运行状态::

    $ ./bin/emqx_ctl status

    Node 'emqx@127.0.0.1' is started
    emqx 2.1.0 is running

----------
broker命令
----------

broker命令查询服务器基本信息，启动时间，统计数据与性能数据。

+----------------+-----------------------------------------------+
| broker         | 查询EMQ消息服务器描述、版本、启动时间         |
+----------------+-----------------------------------------------+
| broker pubsub  | 查询核心的Erlang PubSub进程状态(调试)         |
+----------------+-----------------------------------------------+
| broker stats   | 查询连接(Client)、会话(Session)、主题(Topic)、|
|                | 订阅(Subscription)、路由(Route)统计信息       |
+----------------+-----------------------------------------------+
| broker metrics | 查询MQTT报文(Packet)、消息(Message)收发统计   |
+----------------+-----------------------------------------------+

查询EMQ X服务器基本信息包括版本、启动时间等::

    $ ./bin/emqx_ctl broker

    sysdescr  : EMQ X
    version   : 2.1.0
    uptime    : 3 hours, 56 minutes, 48 seconds
    datetime  : 2017-01-12 23:22:05

broker stats
------------

查询服务器客户端连接(Client)、会话(Session)、主题(Topic)、订阅(Subscription)、路由(Route)统计::

    $ ./bin/emqx_ctl broker stats

    clients/count       : 0
    clients/max         : 0
    retained/count      : 3
    retained/max        : 3
    routes/count        : 0
    routes/max          : 0
    sessions/count      : 0
    sessions/max        : 0
    subscribers/count   : 0
    subscribers/max     : 0
    subscriptions/count : 0
    subscriptions/max   : 0
    topics/count        : 0
    topics/max          : 0

broker metrics
--------------

查询服务器流量(Bytes)、MQTT报文(Packets)、消息(Messages)收发统计::

    $ ./bin/emqx_ctl broker metrics

    bytes/received          : 383
    bytes/sent              : 108
    messages/dropped        : 3
    messages/qos0/received  : 0
    messages/qos0/sent      : 3
    messages/qos1/received  : 0
    messages/qos1/sent      : 0
    messages/qos2/dropped   : 0
    messages/qos2/received  : 6
    messages/qos2/sent      : 0
    messages/received       : 6
    messages/retained       : 3
    messages/sent           : 3
    packets/connack         : 7
    packets/connect         : 7
    packets/disconnect      : 6
    packets/pingreq         : 0
    packets/pingresp        : 0
    packets/puback/missed   : 0
    packets/puback/received : 0
    packets/puback/sent     : 0
    packets/pubcomp/missed  : 0
    packets/pubcomp/received: 0
    packets/pubcomp/sent    : 6
    packets/publish/received: 6
    packets/publish/sent    : 3
    packets/pubrec/missed   : 0
    packets/pubrec/received : 0
    packets/pubrec/sent     : 6
    packets/pubrel/missed   : 0
    packets/pubrel/received : 6
    packets/pubrel/sent     : 0
    packets/received        : 26
    packets/sent            : 23
    packets/suback          : 1
    packets/subscribe       : 1
    packets/unsuback        : 0
    packets/unsubscribe     : 0

-----------
cluster命令
-----------

cluster命令集群多个EMQ消息服务器节点(进程):

+-----------------------+---------------------+
| cluster join <Node>   | 加入集群            |
+-----------------------+---------------------+
| cluster leave         | 离开集群            |
+-----------------------+---------------------+
| cluster remove <Node> | 从集群删除节点      |
+-----------------------+---------------------+
| cluster status        | 查询集群状态        |
+-----------------------+---------------------+

cluster命令集群本机两个EMQ节点示例:

+-----------+---------------------+-------------+
| 目录      | 节点名              | MQTT端口    |
+-----------+---------------------+-------------+
| emqx1     | emqx1@127.0.0.1     | 1883        |
+-----------+---------------------+-------------+
| emqx2     | emqx2@127.0.0.1     | 2883        |
+-----------+---------------------+-------------+

启动emqx1::

    cd emqx1 && ./bin/emqx start

启动emqx2::

    cd emqx2 && ./bin/emqx start

emqx2节点与emqx1集群，emqx2目录下::

    $ ./bin/emqx_ctl cluster join emqx1@127.0.0.1

    Join the cluster successfully.
    Cluster status: [{running_nodes,['emqx1@127.0.0.1','emqx2@127.0.0.1']}]

任意节点目录下查询集群状态::

    $ ./bin/emqx_ctl cluster status

    Cluster status: [{running_nodes,['emqx2@127.0.0.1','emqx1@127.0.0.1']}]

集群消息路由测试::

    # emqx1节点上订阅x
    mosquitto_sub -t x -q 1 -p 1883

    # emqx2节点上向x发布消息
    mosquitto_pub -t x -q 1 -p 2883 -m hello

emqx2节点离开集群::

    cd emqx2 && ./bin/emqx_ctl cluster leave

emqx1节点下删除emqx2::

    cd emqx1 && ./bin/emqx_ctl cluster remove emqx2@127.0.0.1

-----------
clients命令
-----------

clients命令查询连接的MQTT客户端。

+-------------------------+-----------------------------+
| clients list            | 查询全部客户端连接          |
+-------------------------+-----------------------------+
| clients show <ClientId> | 根据ClientId查询客户端      |
+-------------------------+-----------------------------+
| clients kick <ClientId> | 根据ClientId踢出客户端      |
+-------------------------+-----------------------------+

clients list
------------

查询全部客户端连接::

    $ ./bin/emqx_ctl clients list

    Client(mosqsub/43832-airlee.lo, clean_sess=true, username=test, peername=127.0.0.1:64896, connected_at=1452929113)
    Client(mosqsub/44011-airlee.lo, clean_sess=true, username=test, peername=127.0.0.1:64961, connected_at=1452929275)
    ...

返回Client对象的属性:

+--------------+-----------------------------+
| clean_sess   | 清除会话标记                |
+--------------+-----------------------------+
| username     | 用户名                      |
+--------------+-----------------------------+
| peername     | 对端TCP地址                 |
+--------------+-----------------------------+
| connected_at | 客户端连接时间              |
+--------------+-----------------------------+

clients show <ClientId>
-----------------------

根据ClientId查询客户端::

    ./bin/emqx_ctl clients show "mosqsub/43832-airlee.lo"

    Client(mosqsub/43832-airlee.lo, clean_sess=true, username=test, peername=127.0.0.1:64896, connected_at=1452929113)

clients kick <ClientId>
-----------------------

根据ClientId踢出客户端::

    ./bin/emqx_ctl clients kick "clientid"

------------
sessions命令
------------

sessions命令查询MQTT连接会话。EMQ X会为每个连接创建会话，clean_session标记true，创建临时(transient)会话；clean_session标记为false，创建持久会话(persistent)。

+--------------------------+-----------------------------+
| sessions list            | 查询全部会话                |
+--------------------------+-----------------------------+
| sessions list persistent | 查询全部持久会话            |
+--------------------------+-----------------------------+
| sessions list transient  | 查询全部临时会话            |
+--------------------------+-----------------------------+
| sessions show <ClientId> | 根据ClientID查询会话        |
+--------------------------+-----------------------------+

sessions list
-------------

查询全部会话::

    $ ./bin/emqx_ctl sessions list

    Session(clientid, clean_sess=false, max_inflight=100, inflight_queue=0, message_queue=0, message_dropped=0, awaiting_rel=0, awaiting_ack=0, awaiting_comp=0, created_at=1452935508)
    Session(mosqsub/44101-airlee.lo, clean_sess=true, max_inflight=100, inflight_queue=0, message_queue=0, message_dropped=0, awaiting_rel=0, awaiting_ack=0, awaiting_comp=0, created_at=1452935401)

返回Session对象属性:

+-------------------+------------------------------------+
| clean_sess        | false: 持久会话，true: 临时会话    |
+-------------------+------------------------------------+
| max_inflight      | 飞行窗口(最大允许同时下发消息数)   |
+-------------------+------------------------------------+
| inflight_queue    | 当前正在下发的消息数               |
+-------------------+------------------------------------+
| message_queue     | 当前缓存消息数                     |
+-------------------+------------------------------------+
| message_dropped   | 会话丢掉的消息数                   |
+-------------------+------------------------------------+
| awaiting_rel      | 等待客户端发送PUBREL的QoS2消息数   |
+-------------------+------------------------------------+
| awaiting_ack      | 等待客户端响应PUBACK的QoS1/2消息数 |
+-------------------+------------------------------------+
| awaiting_comp     | 等待客户端响应PUBCOMP的QoS2消息数  |
+-------------------+------------------------------------+
| created_at        | 会话创建时间戳                     |
+-------------------+------------------------------------+

sessions list persistent
------------------------

查询全部持久会话::

    $ ./bin/emqx_ctl sessions list persistent

    Session(clientid, clean_sess=false, max_inflight=100, inflight_queue=0, message_queue=0, message_dropped=0, awaiting_rel=0, awaiting_ack=0, awaiting_comp=0, created_at=1452935508)

sessions list transient
-----------------------

查询全部临时会话::

    $ ./bin/emqx_ctl sessions list transient

    Session(mosqsub/44101-airlee.lo, clean_sess=true, max_inflight=100, inflight_queue=0, message_queue=0, message_dropped=0, awaiting_rel=0, awaiting_ack=0, awaiting_comp=0, created_at=1452935401)

sessions show <ClientId>
------------------------

根据ClientId查询会话::

    $ ./bin/emqx_ctl sessions show clientid

    Session(clientid, clean_sess=false, max_inflight=100, inflight_queue=0, message_queue=0, message_dropped=0, awaiting_rel=0, awaiting_ack=0, awaiting_comp=0, created_at=1452935508)

----------
routes命令
----------

routes命令查询路由表。

routes list
-----------

查询全部路由::

    $ ./bin/emqx_ctl routes list

    t2/# -> emqx2@127.0.0.1
    t/+/x -> emqx2@127.0.0.1,emq1@127.0.0.1

routes show <Topic>
-------------------

根据Topic查询一条路由::

    $ ./bin/emqx_ctl routes show t/+/x

    t/+/x -> emqx2@127.0.0.1,emqx1@127.0.0.1

----------
topics命令
----------

topics命令查询当前的主题(Topic)表。

topics list
-----------

查询全部主题(Topic)::

    $ ./bin/emqx_ctl topics list

    $SYS/brokers/emqx@127.0.0.1/metrics/packets/subscribe: static
    $SYS/brokers/emqx@127.0.0.1/stats/subscriptions/max: static
    $SYS/brokers/emqx2@127.0.0.1/stats/subscriptions/count: static
    ...

topics show <Topic>
-------------------

查询某个主题(Topic)::

    $ ./bin/emqx_ctl topics show '$SYS/brokers'

    $SYS/brokers: static

-----------------
subscriptions命令
-----------------

subscriptions命令查询消息服务器的订阅(Subscription)表。

+--------------------------------------------+-------------------------+
| subscriptions list                         | 查询全部订阅            |
+--------------------------------------------+-------------------------+
| subscriptions show <ClientId>              | 查询某个ClientId的订阅  |
+--------------------------------------------+-------------------------+

subscriptions list
------------------

查询全部订阅::

    $ ./bin/emqx_ctl subscriptions list

    mosqsub/91042-airlee.lo -> t/y:1
    mosqsub/90475-airlee.lo -> t/+/x:2

subscriptions show <ClientId>
-----------------------------

查询某个Client的订阅::

    $ ./bin/emqx_ctl subscriptions show 'mosqsub/90475-airlee.lo'

    mosqsub/90475-airlee.lo -> t/+/x:2

-----------
plugins命令
-----------

plugins命令用于加载、卸载、查询插件应用。EMQ X消息服务器通过插件扩展认证、定制功能，插件置于plugins/目录下。

+---------------------------+-------------------------+
| plugins list              | 列出全部插件(Plugin)    |
+---------------------------+-------------------------+
| plugins load <Plugin>     | 加载插件(Plugin)        |
+---------------------------+-------------------------+
| plugins unload <Plugin>   | 卸载插件(Plugin)        |
+---------------------------+-------------------------+

plugins list
------------

列出全部插件::

    $ ./bin/emqx_ctl plugins list

    Plugin(emqx_auth_clientid, version=2.1.0, description=EMQ X Authentication with ClientId/Password, active=false)
    Plugin(emqx_auth_http, version=2.1.0, description=EMQ X Authentication/ACL with HTTP API, active=false)
    Plugin(emqx_auth_ldap, version=2.1.0, description=EMQ X Authentication/ACL with LDAP, active=false)
    Plugin(emqx_auth_mongo, version=2.1.0, description=EMQ X Authentication/ACL with MongoDB, active=false)
    Plugin(emqx_auth_mysql, version=2.1.0, description=EMQ X Authentication/ACL with MySQL, active=false)
    Plugin(emqx_auth_pgsql, version=2.1, description=EMQ X Authentication/ACL with PostgreSQL, active=false)
    Plugin(emqx_auth_redis, version=2.1.0, description=EMQ X Authentication/ACL with Redis, active=false)
    Plugin(emqx_auth_username, version=2.1.0, description=EMQ X Authentication with Username/Password, active=false)
    Plugin(emqx_backend_cassa, version=2.1.0, description=EMQ X Cassandra Backend, active=false)
    Plugin(emqx_backend_mongo, version=2.1.0, description=EMQ X Mongodb Backend, active=false)
    Plugin(emqx_backend_mysql, version=2.1, description=EMQ X MySQL Backend, active=false)
    Plugin(emqx_backend_pgsql, version=2.1.0, description=EMQ X PostgreSQL Backend, active=false)
    Plugin(emqx_backend_redis, version=2.1.0, description=EMQ X Redis Backend, active=false)
    Plugin(emqx_bridge_kafka, version=2.1.0, description=EMQ X Kafka Bridge, active=false)
    Plugin(emqx_bridge_rabbit, version=2.1.0, description=EMQ X Bridge RabbitMQ, active=false)
    Plugin(emqx_dashboard, version=2.1.0, description=EMQ X Dashboard, active=true)
    Plugin(emqx_modules, version=2.1.0, description=EMQ X Modules, active=true)
    Plugin(emqx_recon, version=2.1.0, description=Recon Plugin, active=true)
    Plugin(emqx_reloader, version=2.1, description=Reloader Plugin, active=false)
    Plugin(emqx_retainer, version=2.1, description=EMQ X Retainer, active=true)

插件属性:

+-------------+-----------------+
| version     | 插件版本        |
+-------------+-----------------+
| description | 插件描述        |
+-------------+-----------------+
| active      | 是否已加载      |
+-------------+-----------------+

load <Plugin>
-------------

加载插件::

    $ ./bin/emqx_ctl plugins load emqx_recon

    Start apps: [emqx_recon]
    Plugin emqx_recon loaded successfully.

unload <Plugin>
---------------

卸载插件::

    $ ./bin/emqx_ctl plugins unload emqx_recon

    Plugin emqx_recon unloaded successfully.

-----------
bridges命令
-----------

bridges命令用于在多台EMQ X服务器节点间创建桥接::

                  ---------             ---------
    Publisher --> | node1 | --Bridge--> | node2 | --> Subscriber
                  ---------             ---------

+----------------------------------------+---------------------------+
| bridges list                           | 查询全部桥接              |
+----------------------------------------+---------------------------+
| bridges options                        | 查询创建桥接选项          |
+----------------------------------------+---------------------------+
| bridges start <Node> <Topic>           | 创建桥接                  |
+----------------------------------------+---------------------------+
| bridges start <Node> <Topic> <Options> | 创建桥接并带选项设置      |
+----------------------------------------+---------------------------+
| bridges stop <Node> <Topic>            | 删除桥接                  |
+----------------------------------------+---------------------------+

创建一条emqx1 -> emqx2节点的桥接，转发传感器主题(Topic)消息到emqx2::

    $ ./bin/emqx_ctl bridges start emqx2@127.0.0.1 sensor/#

    bridge is started.

    $ ./bin/emqx_ctl bridges list

    bridge: emqx1@127.0.0.1--sensor/#-->emqx2@127.0.0.1

测试emqx1--sensor/#-->emqx2的桥接::

    #emqx2节点上

    mosquitto_sub -t sensor/# -p 2883 -d

    #emqx1节点上

    mosquitto_pub -t sensor/1/temperature -m "37.5" -d

bridge options
--------------

查询bridge创建选项设置::

    $ ./bin/emqx_ctl bridges options

    Options:
      qos     = 0 | 1 | 2
      prefix  = string
      suffix  = string
      queue   = integer
    Example:
      qos=2,prefix=abc/,suffix=/yxz,queue=1000

bridges stop <Node> <Topic>
---------------------------

删除emqx1--sensor/#-->emqx2的桥接::

    $ ./bin/emqx_ctl bridges stop emqx2@127.0.0.1 sensor/#

    bridge is stopped.

------
vm命令
------

vm命令用于查询Erlang虚拟机负载、内存、进程、IO信息。

+-------------+------------------------+
| vm all      | 查询VM全部信息         |
+-------------+------------------------+
| vm load     | 查询VM负载             |
+-------------+------------------------+
| vm memory   | 查询VM内存             |
+-------------+------------------------+
| vm process  | 查询VM Erlang进程数量  |
+-------------+------------------------+
| vm io       | 查询VM io最大文件句柄  |
+-------------+------------------------+

vm load
-------

查询VM负载::

    $ ./bin/emqx_ctl vm load

    cpu/load1               : 2.21
    cpu/load5               : 2.60
    cpu/load15              : 2.36

vm memory
---------

查询VM内存::

    $ ./bin/emqx_ctl vm memory

    memory/total            : 23967736
    memory/processes        : 3594216
    memory/processes_used   : 3593112
    memory/system           : 20373520
    memory/atom             : 512601
    memory/atom_used        : 491955
    memory/binary           : 51432
    memory/code             : 13401565
    memory/ets              : 1082848

vm process
----------

查询Erlang进程数量::

    $ ./bin/emqx_ctl vm process

    process/limit           : 8192
    process/count           : 221

vm io
-----

查询IO最大句柄数::

    $ ./bin/emqx_ctl vm io

    io/max_fds              : 2560
    io/active_fds           : 1

---------
trace命令
---------

trace命令用于追踪某个客户端或Topic，打印日志信息到文件。

+-----------------------------------+-----------------------------------+
| trace list                        | 查询全部开启的追踪                |
+-----------------------------------+-----------------------------------+
| trace client <ClientId> <LogFile> | 开启Client追踪，日志到文件        |
+-----------------------------------+-----------------------------------+
| trace client <ClientId> off       | 关闭Client追踪                    |
+-----------------------------------+-----------------------------------+
| trace topic <Topic> <LogFile>     | 开启Topic追踪，日志到文件         |
+-----------------------------------+-----------------------------------+
| trace topic <Topic> off           | 关闭Topic追踪                     |
+-----------------------------------+-----------------------------------+

trace client <ClientId> <LogFile>
---------------------------------

开启Client追踪::

    $ ./bin/emqx_ctl trace client clientid log/clientid_trace.log

    trace client clientid successfully.


trace client <ClientId> off
---------------------------

关闭Client追踪::

    $ ./bin/emqx_ctl trace client clientid off

    stop to trace client clientid successfully.

trace topic <Topic> <LogFile>
-----------------------------

开启Topic追踪::

    $ ./bin/emqx_ctl trace topic topic log/topic_trace.log

    trace topic topic successfully.

trace topic <Topic> off
-----------------------

关闭Topic追踪::

    $ ./bin/emqx_ctl trace topic topic off

    stop to trace topic topic successfully.

trace list
----------

查询全部开启的追踪::

    $ ./bin/emqx_ctl trace list

    trace client clientid -> log/clientid_trace.log
    trace topic topic -> log/topic_trace.log

---------
listeners
---------

listeners命令用于查询开启的TCP服务监听器::

    $ ./bin/emqx_ctl listeners

    listener on mqtt:wss:8084
      acceptors       : 4
      max_clients     : 64
      current_clients : 0
      shutdown_count  : []
    listener on mqtt:ssl:8883
      acceptors       : 4
      max_clients     : 1024
      current_clients : 0
      shutdown_count  : []
    listener on mqtt:ws:8083
      acceptors       : 4
      max_clients     : 64
      current_clients : 0
      shutdown_count  : []
    listener on mqtt:tcp:0.0.0.0:1883
      acceptors       : 8
      max_clients     : 1024
      current_clients : 1
      shutdown_count  : [{closed,2}]
    listener on mqtt:tcp:127.0.0.1:11883
      acceptors       : 4
      max_clients     : 1024
      current_clients : 0
      shutdown_count  : []
    listener on dashboard:http:18083
      acceptors       : 2
      max_clients     : 512
      current_clients : 0
      shutdown_count  : []

listener参数说明:

+-----------------+-----------------------------------+
| acceptors       | TCP Acceptor池                    |
+-----------------+-----------------------------------+
| max_clients     | 最大允许连接数                    |
+-----------------+-----------------------------------+
| current_clients | 当前连接数                        |
+-----------------+-----------------------------------+
| shutdown_count  | Socket关闭原因统计                |
+-----------------+-----------------------------------+


-----------
license命令
-----------

查询license信息::

    $ ./bin/emqx_ctl license info

    vendor      : EMQ Enterprise, Inc
    product     : EMQ X Enterprise
    expiry      : {{2017,6,30},{0,0,0}}
    customer    : Free Trial
    userdata    : [{max_clients,100}]

重新加载license::

    $ ./bin/emqx_ctl license reload <File>
    ok.

----------
mnesia命令
----------

查询mnesia数据库系统状态。

----------
admins命令
----------

Dashboard插件会自动注册admins命令，用于创建、删除管理员账号，重置管理员密码。

+------------------------------------+-----------------------------+
| admins add <Username> <Password>   | 创建admin账号               |
+------------------------------------+-----------------------------+
| admins passwd <Username> <Password>| 重置admin密码               |
+------------------------------------+-----------------------------+
| admins del <Username>              | 删除admin账号               |
+------------------------------------+-----------------------------+

admins add
----------

创建admin账户::

    $ ./bin/emqx_ctl admins add root public
    ok

admins passwd
-------------

重置admin账户密码::

    $ ./bin/emqx_ctl admins passwd root private
    ok

admins del
----------

删除admin账户::

    $ ./bin/emqx_ctl admins del root
    ok

