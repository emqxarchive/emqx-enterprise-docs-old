
.. _plugins:

扩展插件 (Plugins)
^^^^^^^^^^^^^^^^^^^

*EMQ X* 消息服务器通过模块注册和钩子(Hooks)机制，支持用户开发扩展插件定制服务器认证鉴权与业务功能。

*EMQ X* 官方提供的插件包括：

+---------------------------+---------------------------------------+---------------------------+
| 插件                      | 配置文件                              | 说明                      |
+===========================+=======================================+===========================+
| `emqx_dashboard`_         + etc/plugins/emqx_dashbord.conf        | Web 控制台插件(默认加载)  |
+---------------------------+---------------------------------------+---------------------------+
| `emqx_psk_file`_          + etc/plugins/emqx_psk_file.conf        | PSK 支持                  |
+---------------------------+---------------------------------------+---------------------------+
| `emqx_web_hook`_          + etc/plugins/emqx_web_hook.conf        | Web Hook 插件             |
+---------------------------+---------------------------------------+---------------------------+
| `emqx_lua_hook`_          + etc/plugins/emqx_lua_hook.conf        | Lua Hook 插件             |
+---------------------------+---------------------------------------+---------------------------+
| `emqx_retainer`_          + etc/plugins/emqx_retainer.conf        | Retain 消息存储模块       |
+---------------------------+---------------------------------------+---------------------------+
| `emqx_rule_engine`_       + etc/plugins/emqx_rule_engine.conf     | 规则引擎                  |
+---------------------------+---------------------------------------+---------------------------+
| `emqx_delayed_publish`_   + etc/plugins/emqx_delayed_publish.conf | 客户端延时发布消息支持    |
+---------------------------+---------------------------------------+---------------------------+
| `emqx_coap`_              + etc/plugins/emqx_coap.conf            | CoAP 协议支持             |
+---------------------------+---------------------------------------+---------------------------+
| `emqx_lwm2m`_             + etc/plugins/emqx_lwm2m.conf           | LwM2M 协议支持            |
+---------------------------+---------------------------------------+---------------------------+
| `emqx_sn`_                + etc/plugins/emqx_sn.conf              | MQTT-SN 协议支持          |
+---------------------------+---------------------------------------+---------------------------+
| `emqx_stomp`_             + etc/plugins/emqx_stomp.conf           | Stomp 协议支持            |
+---------------------------+---------------------------------------+---------------------------+
| `emqx_recon`_             + etc/plugins/emqx_recon.conf           | Recon 性能调试            |
+---------------------------+---------------------------------------+---------------------------+
| `emqx_reloader`_          + etc/plugins/emqx_reloader.conf        | Reloader 代码热加载插件   |
+---------------------------+---------------------------------------+---------------------------+
| `emqx_plugin_template`_   + etc/plugins/emqx_plugin_template.conf | 插件开发模版              |
+---------------------------+---------------------------------------+---------------------------+

其中插件的加载有四种方式：

1. 默认加载
2. 命令行启停插件
3. 使用 Dashboard 启停插件
4. 调用管理 API 启停插件

**开启默认加载**

如需在系统启动时就默认启动某插件，则直接在 ``data/loaded_plugins`` 配置入需要启动的插件，例如默认开启的加载的插件有：

.. code:: erlang

    emqx_management.
    emqx_rule_engine.
    emqx_recon.
    emqx_retainer.
    emqx_dashboard.

**命令行启停插件**

在运行过程中，我们可以通过 CLI 命令的方式查看可用的插件列表、和启停某插件：

.. code:: bash

    ## 显示所有可用的插件列表
    ./bin/emqx_ctl plugins list

    ## 加载某插件
    ./bin/emqx_ctl plugins load emqx_auth_username

    ## 卸载某插件
    ./bin/emqx_ctl plugins unload emqx_auth_username

    ## 重新加载某插件
    ./bin/emqx_ctl plugins reload emqx_auth_username

**使用 Dashboard 启停插件**

如果 *EMQ X* 开启了 Dashbord 的插件(默认开启) 还可以直接通过访问 ``http://localhost:18083/plugins`` 中的插件管理页面启停、或者配置插件。

Dashboard 插件
-----------------

`emqx_dashboard`_ 是 *EMQ X* 消息服务器的 Web 管理控制台, 该插件默认开启。当 *EMQ X* 启动成功后，可访问 ``http://localhost:18083`` 进行查看，默认用户名/密码: admin/public。

Dashboard 中可查询 *EMQ X* 消息服务器基本信息、统计数据、负载情况，查询当前客户端列表(Connections)、会话(Sessions)、路由表(Topics)、订阅关系(Subscriptions) 等详细信息。

.. image:: ./_static/images/dashboard.png

除此之外，Dashboard 默认提供了一系列的 REST API 供前端调用。其详情可以参考 ``Dashboard -> HTTP API`` 部分。

Dashboard 插件设置
::::::::::::::::::

etc/plugins/emqx_dashboard.conf:

.. code:: properties

    ## Dashboard 默认用户名/密码
    dashboard.default_user.login = admin
    dashboard.default_user.password = public

    ## Dashboard HTTP 服务端口配置
    dashboard.listener.http = 18083
    dashboard.listener.http.acceptors = 2
    dashboard.listener.http.max_clients = 512

    ## Dashboard HTTPS 服务端口配置
    ## dashboard.listener.https = 18084
    ## dashboard.listener.https.acceptors = 2
    ## dashboard.listener.https.max_clients = 512
    ## dashboard.listener.https.handshake_timeout = 15s
    ## dashboard.listener.https.certfile = etc/certs/cert.pem
    ## dashboard.listener.https.keyfile = etc/certs/key.pem
    ## dashboard.listener.https.cacertfile = etc/certs/cacert.pem
    ## dashboard.listener.https.verify = verify_peer
    ## dashboard.listener.https.fail_if_no_peer_cert = true

PSK 认证插件
--------------

`emqx_psk_file`_ 插件主要提供了 PSK 支持。其目的是用于在客户端建立 TLS/DTLS 连接时，通过 PSK 方式实现 **连接认证** 的功能。

配置 PSK 认证插件
:::::::::::::::::

etc/plugins/emqx_psk_file.conf:

.. code:: properties

    psk.file.path = etc/psk.txt

WebHook 插件
--------------

`emqx_web_hook`_ 插件可以将所有 *EMQ X* 的事件及消息都发送到指定的 HTTP 服务器。

配置 WebHook 插件
:::::::::::::::::

etc/plugins/emqx_web_hook.conf:

.. code:: properties

    ## 回调的 Web Server 地址
    web.hook.api.url = http://127.0.0.1:8080

    ## 编码 Payload 字段
    ## 枚举值: undefined | base64 | base62
    ## 默认值: undefined (不进行编码)
    ## web.hook.encode_payload = base64

    ## 消息、事件配置
    web.hook.rule.client.connected.1     = {"action": "on_client_connected"}
    web.hook.rule.client.disconnected.1  = {"action": "on_client_disconnected"}
    web.hook.rule.client.subscribe.1     = {"action": "on_client_subscribe"}
    web.hook.rule.client.unsubscribe.1   = {"action": "on_client_unsubscribe"}
    web.hook.rule.session.created.1      = {"action": "on_session_created"}
    web.hook.rule.session.subscribed.1   = {"action": "on_session_subscribed"}
    web.hook.rule.session.unsubscribed.1 = {"action": "on_session_unsubscribed"}
    web.hook.rule.session.terminated.1   = {"action": "on_session_terminated"}
    web.hook.rule.message.publish.1      = {"action": "on_message_publish"}
    web.hook.rule.message.deliver.1      = {"action": "on_message_deliver"}
    web.hook.rule.message.acked.1        = {"action": "on_message_acked"}

Lua 插件
-----------

`emqx_lua_hook`_ 插件将所有的事件和消息都发送到指定的 Lua 函数上。其具体使用参见其 README。

Retainer 插件
---------------

`emqx_retainer`_ 该插件设置为默认启动，为 *EMQ X* 提供 Retained 类型的消息支持。它会将所有主题的 Retained 消息存储在集群的数据库中，并待有客户端订阅该主题的时候将该消息投递出去。

配置 Retainer 插件
::::::::::::::::::

etc/plugins/emqx_retainer.conf:

.. code:: properties

    ## retained 消息存储方式
    ##  - ram: 仅内存
    ##  - disc: 内存和磁盘
    ##  - disc_only: 仅磁盘
    retainer.storage_type = ram

    ## 最大存储数 (0表示未限制)
    retainer.max_retained_messages = 0

    ## 单条最大可存储消息大小
    retainer.max_payload_size = 1MB

    ## 过期时间, 0 表示永不过期
    ## 单位: h 小时; m 分钟; s 秒。如 60m 表示 60 分钟
    retainer.expiry_interval = 0

Delayed Publish 插件
-----------------------

`emqx_delayed_publish`_ 提供了延迟发送消息的功能。当客户端使用特殊主题前缀 ``$delayed/<seconds>/`` 发布消息到 *EMQ X* 时，*EMQ X* 将在 ``<seconds>`` 秒后发布该主题消息。

CoAP 协议插件
----------------

`emqx_coap`_ 提供对 CoAP 协议(RFC 7252)的支持。

配置 CoAP 协议插件
::::::::::::::::::

etc/plugins/emqx_coap.conf:

.. code:: properties

    coap.port = 5683

    coap.keepalive = 120s

    coap.enable_stats = off

若开启以下配置，则可以支持 DTLS：

.. code:: properties

    ## DTLS 监听端口
    coap.dtls.port = 5684

    coap.dtls.keyfile = {{ platform_etc_dir }}/certs/key.pem

    coap.dtls.certfile = {{ platform_etc_dir }}/certs/cert.pem

    ## 双向认证相关
    ## coap.dtls.verify = verify_peer
    ## coap.dtls.cacertfile = {{ platform_etc_dir }}/certs/cacert.pem
    ## coap.dtls.fail_if_no_peer_cert = false

测试 CoAP 插件
::::::::::::::

我们可以通过安装 `libcoap`_ 来测试 *EMQ X* 对 CoAP 协议的支持情况。

.. code:: bash

    yum install libcoap

    % coap client publish message
    coap-client -m put -e "qos=0&retain=0&message=payload&topic=hello" coap://localhost/mqtt

LwM2M 协议插件
----------------

`emqx_lwm2m`_ 提供对 LwM2M 协议的支持。

配置 LwM2M 插件
:::::::::::::::

etc/plugins/emqx_lwm2m.conf:

.. code:: properties

    ## LwM2M 监听端口
    lwm2m.port = 5683

    ## Lifetime 限制
    lwm2m.lifetime_min = 1s
    lwm2m.lifetime_max = 86400s

    ## Q Mode 模式下 `time window` 长度, 单位秒。
    ## 超过该 window 的消息都将被缓存
    #lwm2m.qmode_time_window = 22

    ## LwM2M 是否部署在 coaproxy 后
    #lwm2m.lb = coaproxy

    ## 设备上线后，主动 observe 所有的 objects
    #lwm2m.auto_observe = off

    ## client register 成功后主动向 EMQ X 订阅的主题
    ## 占位符:
    ##    '%e': Endpoint Name
    ##    '%a': IP Address
    lwm2m.topics.command = lwm2m/%e/dn/#

    ## client 应答消息(response) 到 EMQ X 的主题
    lwm2m.topics.response = lwm2m/%e/up/resp

    ## client 通知类消息(noify message) 到 EMQ X 的主题
    lwm2m.topics.notify = lwm2m/%e/up/notify

    ## client 注册类消息(register message) 到 EMQ X 的主题
    lwm2m.topics.register = lwm2m/%e/up/resp

    # client 更新类消息(update message) 到 EMQ X 的主题
    lwm2m.topics.update = lwm2m/%e/up/resp

    # Object 定义的 xml 文件位置
    lwm2m.xml_dir =  etc/lwm2m_xml

同样可以通过以下配置打开 DTLS 支持：

.. code:: properties

    # DTLS 证书配置
    lwm2m.certfile = etc/certs/cert.pem
    lwm2m.keyfile = etc/certs/key.pem

MQTT-SN 协议插件
------------------

`emqx_sn`_ 插件提供对 `MQTT-SN`_ 协议的支持。

配置 MQTT-SN 协议插件
:::::::::::::::::::::

etc/plugins/emqx_sn.conf:

.. code:: properties

    mqtt.sn.port = 1884

Stomp 协议插件
-----------------

`emqx_stomp`_ 提供对 Stomp 协议的支持。支持客户端通过 Stomp 1.0/1.1/1.2 协议连接 EMQ X，发布订阅 MQTT 消息。

配置 Stomp 插件
:::::::::::::::

.. NOTE:: Stomp 协议端口: 61613

etc/plugins/emqx_stomp.conf:

.. code:: properties

    stomp.default_user.login = guest

    stomp.default_user.passcode = guest

    stomp.allow_anonymous = true

    stomp.frame.max_headers = 10

    stomp.frame.max_header_length = 1024

    stomp.frame.max_body_length = 8192

    stomp.listener = 61613

    stomp.listener.acceptors = 4

    stomp.listener.max_clients = 512

Recon 性能调试插件
-------------------

`emqx_recon`_ 插件集成了 recon 性能调测库，可用于查看当前系统的一些状态信息，例如：

.. code:: bash

    ./bin/emqx_ctl recon

    recon memory                 #recon_alloc:memory/2
    recon allocated              #recon_alloc:memory(allocated_types, current|max)
    recon bin_leak               #recon:bin_leak(100)
    recon node_stats             #recon:node_stats(10, 1000)
    recon remote_load Mod        #recon:remote_load(Mod)

配置 Recon 插件
:::::::::::::::

etc/plugins/emqx_recon.conf:

.. code:: properties

    %% Garbage Collection: 10 minutes
    recon.gc_interval = 600

Reloader 热加载插件
--------------------

`emqx_reloader`_ 用于开发调试的代码热升级插件。加载该插件后 *EMQ X* 会根据配置的时间间隔自动热升级更新代码。

同时，也提供了 CLI 命令来指定 reload 某一个模块：

.. code:: bash

    ./bin/emqx_ctl reload <Module>

.. NOTE:: 产品部署环境不建议使用该插件。

配置 Reloader 插件
::::::::::::::::::

etc/plugins/emqx_reloader.conf:

.. code:: properties

    reloader.interval = 60

    reloader.logfile = log/reloader.log

插件开发模版
---------------

`emqx_plugin_template`_ 是一个 *EMQ X* 插件模板，在功能上并无任何意义。

开发者需要自定义插件时，可以查看该插件的代码和结构，以更快地开发一个标准的 *EMQ X* 插件。插件实际是一个普通的 ``Erlang Application``，其配置文件为: ``etc/${PluginName}.config``。

EMQ X R3.1 插件开发
--------------------

创建插件项目
::::::::::::

参考 `emqx_plugin_template`_ 插件模版创建新的插件项目。

.. NOTE:: 在 ``<plugin name>_app.erl`` 文件中必须加上标签 ``-emqx_plugin(?MODULE).`` 以表明这是一个 EMQ X 的插件。

创建认证/访问控制模块
::::::::::::::::::::::

认证演示模块 - emqx_auth_demo.erl

.. code:: erlang

    -module(emqx_auth_demo).

    -export([ init/1
            , check/2
            , description/0
            ]).

    init(Opts) -> {ok, Opts}.

    check(_Credentials = #{client_id := ClientId, username := Username, password := Password}, _State) ->
        io:format("Auth Demo: clientId=~p, username=~p, password=~p~n", [ClientId, Username, Password]),
        ok.

    description() -> "Auth Demo Module".

访问控制演示模块 - emqx_acl_demo.erl

.. code:: erlang

    -module(emqx_acl_demo).

    -include_lib("emqx/include/emqx.hrl").

    %% ACL callbacks
    -export([ init/1
            , check_acl/5
            , reload_acl/1
            , description/0
            ]).

    init(Opts) ->
        {ok, Opts}.

    check_acl({Credentials, PubSub, _NoMatchAction, Topic}, _State) ->
        io:format("ACL Demo: ~p ~p ~p~n", [Credentials, PubSub, Topic]),
        allow.

    reload_acl(_State) ->
        ok.

    description() -> "ACL Demo Module".

注册认证、访问控制模块 - emqx_plugin_template_app.erl

.. code:: erlang

    ok = emqx:hook('client.authenticate', fun emqx_auth_demo:check/2, []),
    ok = emqx:hook('client.check_acl', fun emqx_acl_demo:check_acl/5, []).

注册钩子(Hooks)
::::::::::::::::

通过钩子(Hook)处理客户端上下线、主题订阅、消息收发。

emqx_plugin_template.erl:

.. code:: erlang

    %% Called when the plugin application start
    load(Env) ->
        emqx:hook('client.authenticate', fun ?MODULE:on_client_authenticate/2, [Env]),
        emqx:hook('client.check_acl', fun ?MODULE:on_client_check_acl/5, [Env]),
        emqx:hook('client.connected', fun ?MODULE:on_client_connected/4, [Env]),
        emqx:hook('client.disconnected', fun ?MODULE:on_client_disconnected/3, [Env]),
        emqx:hook('client.subscribe', fun ?MODULE:on_client_subscribe/3, [Env]),
        emqx:hook('client.unsubscribe', fun ?MODULE:on_client_unsubscribe/3, [Env]),
        emqx:hook('session.created', fun ?MODULE:on_session_created/3, [Env]),
        emqx:hook('session.resumed', fun ?MODULE:on_session_resumed/3, [Env]),
        emqx:hook('session.subscribed', fun ?MODULE:on_session_subscribed/4, [Env]),
        emqx:hook('session.unsubscribed', fun ?MODULE:on_session_unsubscribed/4, [Env]),
        emqx:hook('session.terminated', fun ?MODULE:on_session_terminated/3, [Env]),
        emqx:hook('message.publish', fun ?MODULE:on_message_publish/2, [Env]),
        emqx:hook('message.deliver', fun ?MODULE:on_message_deliver/3, [Env]),
        emqx:hook('message.acked', fun ?MODULE:on_message_acked/3, [Env]),
        emqx:hook('message.dropped', fun ?MODULE:on_message_dropped/3, [Env]).

所有可用钩子(Hook)说明:

+------------------------+----------------------------------+
| 钩子                   | 说明                             |
+========================+==================================+
| client.authenticate    | 连接认证                         |
+------------------------+----------------------------------+
| client.check_acl       | ACL 校验                         |
+------------------------+----------------------------------+
| client.connected       | 客户端上线                       |
+------------------------+----------------------------------+
| client.disconnected    | 客户端连接断开                   |
+------------------------+----------------------------------+
| client.subscribe       | 客户端订阅主题                   |
+------------------------+----------------------------------+
| client.unsubscribe     | 客户端取消订阅主题               |
+------------------------+----------------------------------+
| session.created        | 会话创建                         |
+------------------------+----------------------------------+
| session.resumed        | 会话恢复                         |
+------------------------+----------------------------------+
| session.subscribed     | 会话订阅主题后                   |
+------------------------+----------------------------------+
| session.unsubscribed   | 会话取消订阅主题后               |
+------------------------+----------------------------------+
| session.terminated     | 会话终止                         |
+------------------------+----------------------------------+
| message.publish        | MQTT 消息发布                    |
+------------------------+----------------------------------+
| message.deliver        | MQTT 消息进行投递                |
+------------------------+----------------------------------+
| message.acked          | MQTT 消息回执                    |
+------------------------+----------------------------------+
| message.dropped        | MQTT 消息丢弃                    |
+------------------------+----------------------------------+

注册 CLI 命令
:::::::::::::

扩展命令行演示模块 - emqx_cli_demo.erl

.. code:: erlang

    -module(emqx_cli_demo).

    -export([cmd/1]).

    cmd(["arg1", "arg2"]) ->
        emqx_cli:print("ok");

    cmd(_) ->
        emqx_cli:usage([{"cmd arg1 arg2", "cmd demo"}]).

注册命令行模块 - emqx_plugin_template_app.erl

.. code:: erlang

    ok = emqx_ctl:register_command(cmd, {emqx_cli_demo, cmd}, []),

插件加载后，``./bin/emqx_ctl`` 新增命令行：

.. code:: bash

    ./bin/emqx_ctl cmd arg1 arg2

插件配置文件
::::::::::::

插件自带配置文件放置在 ``etc/${plugin_name}.conf|config``。*EMQ X* 支持两种插件配置格式:

1. Erlang 原生配置文件格式 - ``${plugin_name}.config``::

    [
      {plugin_name, [
        {key, value}
      ]}
    ].

2. sysctl 的 ``k = v`` 通用格式 - ``${plugin_name}.conf``::

    plugin_name.key = value

.. NOTE:: ``k = v`` 格式配置需要插件开发者创建 ``priv/plugin_name.schema`` 映射文件。

编译发布插件
::::::::::::

1. clone emqx-rel 项目：

.. code:: bash

    git clone https://github.com/emqx/emqx-rel.git

2. Makefile 增加 ``DEPS``：

.. code:: makefile

    DEPS += plugin_name
    dep_plugin_name = git url_of_plugin

3. relx.config 中 release 段落添加：

.. code:: erlang

    {plugin_name, load},

.. _emqx_dashboard:        https://github.com/emqx/emqx-dashboard
.. _emqx_retainer:         https://github.com/emqx/emqx-retainer
.. _emqx_delayed_publish:  https://github.com/emqx/emqx-delayed-publish
.. _emqx_web_hook:         https://github.com/emqx/emqx-web-hook
.. _emqx_lua_hook:         https://github.com/emqx/emqx-lua-hook
.. _emqx_sn:               https://github.com/emqx/emqx-sn
.. _emqx_coap:             https://github.com/emqx/emqx-coap
.. _emqx_lwm2m:            https://github.com/emqx/emqx-lwm2m
.. _emqx_stomp:            https://github.com/emqx/emqx-stomp
.. _emqx_recon:            https://github.com/emqx/emqx-recon
.. _emqx_reloader:         https://github.com/emqx/emqx-reloader
.. _emqx_psk_file:         https://github.com/emqx/emqx-psk-file
.. _emqx_plugin_template:  https://github.com/emqx/emqx-plugin-template
.. _emqx_rule_engine:      https://github.com/emqx/emqx-rule-engine
.. _recon:                 http://ferd.github.io/recon/
.. _LDAP:                  https://ldap.com
.. _JWT:                   https://jwt.io
.. _libcoap:               https://github.com/obgm/libcoap
.. _MQTT-SN:               https://github.com/emqx/emqx-sn
