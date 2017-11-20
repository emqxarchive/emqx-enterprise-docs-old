
.. _plugins:

========
扩展插件
========

EMQ X消息服务器通过模块注册和钩子(Hooks)机制，扩展插件方式定制认证鉴权、数据存储、桥接转发与管理维护功能。

EMQ X企业版提供的管理维护插件:

+---------------------+-------------------------+----------------+---------------------------+
| 插件                | 配置文件                | 默认加载       | 说明                      |
+=====================+=========================+================+===========================+
| emqx_dashboard      | emqx_dashboard.conf     | 是             | Web控制台插件(默认加载)   |
+---------------------+-------------------------+----------------+---------------------------+
| emqx_modules        | emqx_modules.conf       | 是             | Modules插件               |
+---------------------+-------------------------+----------------+---------------------------+
| emqx_retainer       | emqx_retainer.conf      | 是             | Retain消息存储插件        |
+---------------------+-------------------------+----------------+---------------------------+
| emqx_recon          | emqx_recon.conf         | 是             | Recon性能调试插件         |
+---------------------+-------------------------+----------------+---------------------------+
| emqx_reloader       | emqx_reloader.conf      | 否             | Reloader代码热加载插件    |
+---------------------+-------------------------+----------------+---------------------------+
| emqx_web_hook       | emqx_web_hook.conf      | 否             | Web钩子插件               |
+---------------------+-------------------------+----------------+---------------------------+

-------------
Dashboard插件
-------------

EMQ X Web管理控制台，系统默认加载。URL地址: http://host:18083 ，缺省用户名/密码: admin/public。

控制台可查询 EMQ X消息服务器基本信息、统计数据、度量数据，查询系统客户端(Client)、会话(Session)、主题(Topic)、订阅(Subscription)。

.. image:: ./_static/images/dashboard.png

Dashboard监听器设置
-------------------

配置文件emqx_dashboard.conf，Dashboard监听器默认端口 - 18083:

.. code-block:: properties

    ## HTTP Listener
    dashboard.listener.http = 18083
    dashboard.listener.http.acceptors = 2
    dashboard.listener.http.max_clients = 512

    ## HTTPS Listener
    ## dashboard.listener.https = 18084
    ## dashboard.listener.https.acceptors = 2
    ## dashboard.listener.https.max_clients = 512
    ## dashboard.listener.https.handshake_timeout = 15
    ## dashboard.listener.https.certfile = etc/certs/cert.pem
    ## dashboard.listener.https.keyfile = etc/certs/key.pem
    ## dashboard.listener.https.cacertfile = etc/certs/cacert.pem
    ## dashboard.listener.https.verify = verify_peer
    ## dashboard.listener.https.fail_if_no_peer_cert = true

------------
Retainer插件
------------

Retainer插件负责持久化MQTT Retained消息，配置文件emqx_retainer.conf:

.. code-block:: properties

    ## disc: disc_copies, ram: ram_copies
    ## Notice: retainer's storage_type on each node in a cluster must be the same!
    retainer.storage_type = ram

    ## Max number of retained messages
    retainer.max_message_num = 1000000

    ## Max Payload Size of retained message
    retainer.max_payload_size = 64KB

    ## Expiry interval. Never expired if 0
    ## h - hour
    ## m - minute
    ## s - second
    retainer.expiry_interval = 0

-----------
Modules插件
-----------

Presence、Subscription、Rewrite等扩展模块插件。

Presence扩展模块设置
--------------------

Presence扩展模块向$SYS/主题发布客户端上下线消息:

.. code-block:: properties

    ## Enable Presence, Values: on | off
    module.presence = on

    module.presence.qos = 1

Subscriptions扩展模块设置
-------------------------

Subscription扩展模块支持客户端上线时自动订阅主题:

.. code-block:: properties

    ## Enable Subscription, Values: on | off
    module.subscription = on

    ## Subscribe the Topics automatically when client connected
    module.subscription.1.topic = $client/%c
    ## Qos of the subscription: 0 | 1 | 2
    module.subscription.1.qos = 1

    ## module.subscription.2.topic = $user/%u
    ## module.subscription.2.qos = 1
 
Rewrite扩展模块设置
-------------------

Rewrite扩展模块支持重写发布订阅主题:

.. code-block:: properties

    ## Enable Rewrite, Values: on | off
    module.rewrite = off

    ## {rewrite, Topic, Re, Dest}
    ## module.rewrite.rule.1 = "x/# ^x/y/(.+)$ z/y/$1"
    ## module.rewrite.rule.2 = "y/+/z/# ^y/(.+)/z/(.+)$ y/z/$2"

-----------------
Recon性能调试插件
-----------------

Recon性能调测插件，配置文件emqx_recon.conf，插件支持周期性全局垃圾回收，并向'./bin/emqx_ctl'命令行注册recon命令。

设置全局GC周期
--------------

.. code-block:: properties

    ## Global GC Interval
    ## h - hour
    ## m - minute
    ## s - second
    recon.gc_interval = 5m

Recon插件命令
-------------

.. code-block:: bash

    ./bin/emqx_ctl recon

    recon memory            #recon_alloc:memory/2
    recon allocated         #recon_alloc:memory(allocated_types, current|max)
    recon bin_leak          #recon:bin_leak(100)
    recon node_stats        #recon:node_stats(10, 1000)
    recon remote_load Mod   #recon:remote_load(Mod)

----------------------
Reloader代码热加载插件
----------------------

用于开发调试的代码热升级插件。加载该插件后，EMQ X服务器会自动热升级更新代码。

配置热加载检测周期
-------------------

配置文件emqx_reloader.conf:

.. code-block:: properties

    reloader.interval = 60s

    reloader.logfile = reloader.log

加载Reloader插件
----------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_reloader

Reloader插件命令
----------------

.. code-block:: bash

    ./bin/emqx_ctl reload

    reload <Module>             # Reload a Module

-----------
Web钩子插件
-----------

用于把mqtt消息通过http post方式发送到配置的WebServer

配置Web hook
-------------------

配置文件emqx_web_hook.conf:

.. code-block:: properties

    ## http post web server
    web.hook.api.url = http://127.0.0.1:8080

    ## hook rule
    web.hook.rule.client.connected.1     = {"action": "on_client_connected"}
    web.hook.rule.client.disconnected.1  = {"action": "on_client_disconnected"}
    web.hook.rule.client.subscribe.1     = {"action": "on_client_subscribe"}
    web.hook.rule.client.unsubscribe.1   = {"action": "on_client_unsubscribe"}
    web.hook.rule.session.created.1      = {"action": "on_session_created"}
    web.hook.rule.session.subscribed.1   = {"action": "on_session_subscribed"}
    web.hook.rule.session.unsubscribed.1 = {"action": "on_session_unsubscribed"}
    web.hook.rule.session.terminated.1   = {"action": "on_session_terminated"}
    web.hook.rule.message.publish.1      = {"action": "on_message_publish"}
    web.hook.rule.message.delivered.1    = {"action": "on_message_delivered"}
    web.hook.rule.message.acked.1        = {"action": "on_message_acked"}



加载WebHook插件
----------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_web_hook

.. _recon: http://ferd.github.io/recon/
