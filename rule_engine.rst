
.. _rule_engine:

*********
规则引擎
*********

===================
EMQ X 规则引擎介绍
===================

使用 EMQ X 的规则引擎可以灵活地处理消息和事件。使用规则引擎可以方便地实现诸如将消息转换成指定格式，然后存入数据库表，或者发送到消息队列等。

与 EMQ X 规则引擎相关的概念包括: 规则(rule)、动作(action)、资源(resource) 和 资源类型(resource-type)。

.. sidebar:: 规则、动作、资源的关系

    ::

        规则: {
            SQL 语句,
            动作列表: [
                {
                    动作1,
                    动作参数,
                    绑定资源: {
                        资源配置
                    }
                },
                {
                    动作2,
                    动作参数,
                    绑定资源: {
                        资源配置
                    }
                }, ...
            ]
        }

- 规则 (Rule): 规则由 SQL 语句和动作列表组成。
  SQL 语句用于筛选或转换事件中的数据。
  动作是 SQL 语句匹配通过之后，所执行的任务。动作列表包含一个或多个动作及其参数。
- 动作 (Action): 动作定义了一个针对数据的操作。
  动作可以绑定资源，也可以不绑定。例如，“inspect” 动作不需要绑定资源，它只是简单打印数据内容和动作参数。而 “data_to_webserver” 动作需要绑定一个 web_hook 类型的资源，此资源中配置了 URL。
- 资源 (Resource): 资源是通过资源类型为模板实例化出来的对象，保存了与资源相关的配置(比如数据库连接地址和端口、用户名和密码等)。
- 资源类型 (Resource Type): 资源类型是资源的静态定义，描述了此类型资源需要的配置项。

.. important:: 动作和资源类型是由 emqx 或插件的代码提供的，不能通过 API 和 CLI 动态创建。

.. _rule_sql:

===========
SQL 语句
===========

.. _rule_sql.syntax:

SQL 语法
----------

SQL 语句用于从原始数据中，根据条件筛选出字段，并进行预处理和转换，基本格式为::

    SELECT <字段名> FROM <触发事件> [WHERE <条件>]

FROM、SELECT 和 WHERE 子句:

- ``FROM`` 子句将规则挂载到某个触发事件上，比如 “消息发布”，“连接完成”，“连接断开” 等
- ``SELECT`` 子句用于筛选或转换事件中的字段
- ``WHERE`` 子句用于根据条件筛选事件

.. _rule_sql.examples:

SQL 语句示例:
--------------

- 从 topic 为 "t/a" 的消息中提取所有字段::

    SELECT * FROM "message.publish" WHERE topic = 't/a'

- 从 topic 能够匹配到 't/#' 的消息中提取所有字段。注意这里使用了 **'=~'** 操作符进行带通配符的 topic 匹配::

    SELECT * FROM "message.publish" WHERE topic =~ 't/#'

- 从 topic 能够匹配到 't/#' 的消息中提取 qos，username 和 client_id 字段::

    SELECT qos, username, client_id FROM "message.publish" WHERE topic =~ 't/#'

- 从任意 topic 的消息中提取 username 字段，并且筛选条件为 username = 'Steven'::

    SELECT username FROM "message.publish" WHERE username='Steven'

- 从任意 topic 的消息的消息体(payload) 中提取 x 字段，并创建别名 x 以便在 WHERE 子句中使用。WHERE 子句限定条件为 x = 1。注意 payload 必须为 JSON 格式。举例：此 SQL 语句可以匹配到消息体 {"x": 1}, 但不能匹配到消息体 {"x": 2}::

    SELECT payload.x as x FROM "message.publish" WHERE x=1

- 类似于上面的 SQL 语句，但嵌套地提取消息体中的数据，此 SQL 语句可以匹配到消息体 {"x": {"y": 1}}::

    SELECT payload.x.y as a FROM "message.publish" WHERE a=1

- 在 client_id = 'c1' 尝试连接时，提取其来源 IP 地址和端口号::

    SELECT peername as ip_port FROM "client.connected" WHERE client_id = 'c1'

- 筛选所有订阅 't/#' 主题且订阅级别为 QoS1 的 client_id。注意这里用的是严格相等操作符 **'='**，所以不会匹配主题为 't' 或 't/+/a' 的订阅请求::

    SELECT client_id FROM "client.subscribe" WHERE topic = 't/#' and qos = 1

- 事实上，上例中的 topic 和 qos 字段，是当订阅请求里只包含了一对 (Topic, QoS) 时，为使用方便而设置的别名。但如果订阅请求中 Topic Filters 包含了多个 (Topic, QoS) 组合对，那么必须显式使用 contains_topic() 或 contains_topic_match() 函数来检查 Topic Filters 是否包含指定的 (Topic, QoS)::

    SELECT client_id FROM "client.subscribe" WHERE contains_topic(topic_filters, 't/#')

    SELECT client_id FROM "client.subscribe" WHERE contains_topic(topic_filters, 't/#', 1)

.. important::
    - FROM 子句后面的触发事件需要用双引号 ``""`` 引起来。
    - WHERE 子句后面接筛选条件，如果使用到字符串需要用单引号 ``''`` 引起来。
    - SELECT 子句中，若使用 ``"."`` 符号对 payload 进行嵌套选择，必须保证 payload 为 JSON 格式。

.. _rule_sql.events:

FROM 子句可用的触发事件
------------------------

+---------------------+----------+
|       事件名        |   释义   |
+=====================+==========+
| message.publish     | 消息发布 |
+---------------------+----------+
| message.deliver     | 消息投递 |
+---------------------+----------+
| message.acked       | 消息确认 |
+---------------------+----------+
| message.dropped     | 消息丢弃 |
+---------------------+----------+
| client.connected    | 连接完成 |
+---------------------+----------+
| client.disconnected | 连接断开 |
+---------------------+----------+
| client.subscribe    | 订阅     |
+---------------------+----------+
| client.unsubscribe  | 取消订阅 |
+---------------------+----------+

.. _rule_sql.columns:

SELECT 子句可用的字段
----------------------

SELECT 子句可用的字段与触发事件的类型相关。其中 ``client_id``, ``username`` 和 ``event`` 是通用字段，每种事件类型都有。

message.publish
^^^^^^^^^^^^^^^

+-----------+------------------------------------+
| client_id | Client ID                          |
+-----------+------------------------------------+
| username  | 用户名                             |
+-----------+------------------------------------+
| event     | 事件类型，固定为 "message.publish" |
+-----------+------------------------------------+
| id        | MQTT 消息 ID                       |
+-----------+------------------------------------+
| topic     | MQTT 主题                          |
+-----------+------------------------------------+
| payload   | MQTT 消息体                        |
+-----------+------------------------------------+
| peername  | 客户端的 IPAddress 和 Port         |
+-----------+------------------------------------+
| qos       | MQTT 消息的 QoS                    |
+-----------+------------------------------------+
| timestamp | 时间戳                             |
+-----------+------------------------------------+

message.deliver
^^^^^^^^^^^^^^^

+-------------+------------------------------------+
| client_id   | Client ID                          |
+-------------+------------------------------------+
| username    | 用户名                             |
+-------------+------------------------------------+
| event       | 事件类型，固定为 "message.deliver" |
+-------------+------------------------------------+
| id          | MQTT 消息 ID                       |
+-------------+------------------------------------+
| topic       | MQTT 主题                          |
+-------------+------------------------------------+
| payload     | MQTT 消息体                        |
+-------------+------------------------------------+
| peername    | 客户端的 IPAddress 和 Port         |
+-------------+------------------------------------+
| qos         | MQTT 消息的 QoS                    |
+-------------+------------------------------------+
| timestamp   | 时间戳                             |
+-------------+------------------------------------+
| auth_result | 认证结果                           |
+-------------+------------------------------------+
| mountpoint  | 消息主题挂载点                     |
+-------------+------------------------------------+

message.acked
^^^^^^^^^^^^^

+-----------+----------------------------------+
| client_id | Client ID                        |
+-----------+----------------------------------+
| username  | 用户名                           |
+-----------+----------------------------------+
| event     | 事件类型，固定为 "message.acked" |
+-----------+----------------------------------+
| id        | MQTT 消息 ID                     |
+-----------+----------------------------------+
| topic     | MQTT 主题                        |
+-----------+----------------------------------+
| payload   | MQTT 消息体                      |
+-----------+----------------------------------+
| peername  | 客户端的 IPAddress 和 Port       |
+-----------+----------------------------------+
| qos       | MQTT 消息的 QoS                  |
+-----------+----------------------------------+
| timestamp | 时间戳                           |
+-----------+----------------------------------+

message.dropped
^^^^^^^^^^^^^^^

+-----------+------------------------------------+
| client_id | Client ID                          |
+-----------+------------------------------------+
| username  | 用户名                             |
+-----------+------------------------------------+
| event     | 事件类型，固定为 "message.dropped" |
+-----------+------------------------------------+
| id        | MQTT 消息 ID                       |
+-----------+------------------------------------+
| topic     | MQTT 主题                          |
+-----------+------------------------------------+
| payload   | MQTT 消息体                        |
+-----------+------------------------------------+
| peername  | 客户端的 IPAddress 和 Port         |
+-----------+------------------------------------+
| qos       | MQTT 消息的 QoS                    |
+-----------+------------------------------------+
| timestamp | 时间戳                             |
+-----------+------------------------------------+
| node      | 节点名                             |
+-----------+------------------------------------+

client.connected
^^^^^^^^^^^^^^^^

+--------------+-------------------------------------+
| client_id    | Client ID                           |
+--------------+-------------------------------------+
| username     | 用户名                              |
+--------------+-------------------------------------+
| event        | 事件类型，固定为 "client.connected" |
+--------------+-------------------------------------+
| auth_result  | 认证结果                            |
+--------------+-------------------------------------+
| clean_start  | MQTT clean start 标志位             |
+--------------+-------------------------------------+
| connack      | MQTT CONNACK 结果                   |
+--------------+-------------------------------------+
| connected_at | 连接时间戳                          |
+--------------+-------------------------------------+
| is_bridge    | 是否是桥接                          |
+--------------+-------------------------------------+
| keepalive    | MQTT 保活间隔                       |
+--------------+-------------------------------------+
| mountpoint   | 消息主题挂载点                      |
+--------------+-------------------------------------+
| peername     | 客户端的 IPAddress 和 Port          |
+--------------+-------------------------------------+
| proto_ver    | MQTT 协议版本                       |
+--------------+-------------------------------------+

client.disconnected
^^^^^^^^^^^^^^^^^^^

+-------------+----------------------------------------+
| client_id   | Client ID                              |
+-------------+----------------------------------------+
| username    | 用户名                                 |
+-------------+----------------------------------------+
| event       | 事件类型，固定为 "client.disconnected" |
+-------------+----------------------------------------+
| auth_result | 认证结果                               |
+-------------+----------------------------------------+
| mountpoint  | 消息主题挂载点                         |
+-------------+----------------------------------------+
| peername    | 客户端的 IPAddress 和 Port             |
+-------------+----------------------------------------+
| reason_code | 断开原因码                             |
+-------------+----------------------------------------+

client.subscribe
^^^^^^^^^^^^^^^^

+---------------+-------------------------------------+
| client_id     | Client ID                           |
+---------------+-------------------------------------+
| username      | 用户名                              |
+---------------+-------------------------------------+
| event         | 事件类型，固定为 "client.subscribe" |
+---------------+-------------------------------------+
| auth_result   | 认证结果                            |
+---------------+-------------------------------------+
| mountpoint    | 消息主题挂载点                      |
+---------------+-------------------------------------+
| peername      | 客户端的 IPAddress 和 Port          |
+---------------+-------------------------------------+
| topic_filters | MQTT 订阅列表                       |
+---------------+-------------------------------------+
| topic         | MQTT 订阅列表中的第一个订阅的主题   |
+---------------+-------------------------------------+
| Qos           | MQTT 订阅列表中的第一个订阅的 QoS   |
+---------------+-------------------------------------+

client.unsubscribe
^^^^^^^^^^^^^^^^^^

+---------------+---------------------------------------+
| client_id     | Client ID                             |
+---------------+---------------------------------------+
| username      | 用户名                                |
+---------------+---------------------------------------+
| event         | 事件类型，固定为 "client.unsubscribe" |
+---------------+---------------------------------------+
| auth_result   | 认证结果                              |
+---------------+---------------------------------------+
| mountpoint    | 消息主题挂载点                        |
+---------------+---------------------------------------+
| peername      | 客户端的 IPAddress 和 Port            |
+---------------+---------------------------------------+
| topic_filters | MQTT 订阅列表                         |
+---------------+---------------------------------------+
| topic         | MQTT 订阅列表中的第一个订阅的主题     |
+---------------+---------------------------------------+
| QoS           | MQTT 订阅列表中的第一个订阅的 QoS     |
+---------------+---------------------------------------+

.. _rule_sql.test:

在 Dashboard 中测试 SQL 语句
------------------------------

Dashboard 界面提供了 SQL 语句测试功能，通过给定的 SQL 语句和事件参数，展示 SQL 测试结果。

1. 在创建规则界面，输入 **规则SQL**，并启用 **SQL 测试** 开关:

   .. image:: ./_static/images/sql-test-1@2x.png

2. 修改模拟事件的字段，或者使用默认的配置，点击 **测试** 按钮:

   .. image:: ./_static/images/sql-test-2@2x.png

3. SQL 处理后的结果将在 **测试输出** 文本框里展示:

   .. image:: ./_static/images/sql-test-3@2x.png

============================
规则引擎管理命令和 HTTP API
============================

.. _rule_engine.cli:

规则引擎(rule engine) 命令
----------------------------

rules 命令
^^^^^^^^^^^^^

+------------------------------------------------------+----------------+
| rules list                                           | List all rules |
+------------------------------------------------------+----------------+
| rules show <RuleId>                                  | Show a rule    |
+------------------------------------------------------+----------------+
| emqx_ctl rules create <sql> <actions> [-d [<descr>]] | Create a rule  |
+------------------------------------------------------+----------------+
| rules delete <RuleId>                                | Delete a rule  |
+------------------------------------------------------+----------------+

rules create
""""""""""""

创建一个新的规则。参数:

- <sql>: 规则 SQL
- <actions>: JSON 格式的动作列表
- -d <descr>: 可选，规则描述信息

使用举例::

    ## 创建一个测试规则，简单打印所有发送到 't/a' 主题的消息内容
    $ ./bin/emqx_ctl rules create \
      'select * from "message.publish"' \
      '[{"name":"inspect", "params": {"a": 1}}]' \
      -d 'Rule for debug'

    Rule rule:9a6a725d created

上例创建了一个 ID 为 ``rule:9a6a725d`` 的规则，动作列表里只有一个动作：动作名为 inspect，动作的参数是 ``{"a": 1}``。

rules list
""""""""""

列出当前所有的规则::

    $ ./bin/emqx_ctl rules list

    rule(id='rule:9a6a725d', for='['message.publish']', rawsql='select * from "message.publish"', actions=[{"metrics":...,"name":"inspect","params":...}], metrics=..., enabled='true', description='Rule for debug')

rules show
""""""""""

查询规则::

    ## 查询 RuleID 为 'rule:9a6a725d' 的规则
    $ ./bin/emqx_ctl rules show 'rule:9a6a725d'

    rule(id='rule:9a6a725d', for='['message.publish']', rawsql='select * from "message.publish"', actions=[{"metrics":...,"name":"inspect","params":...}], metrics=..., enabled='true', description='Rule for debug')

rules delete
""""""""""""

删除规则::

    ## 删除 RuleID 为 'rule:9a6a725d' 的规则
    $ ./bin/emqx_ctl rules delete 'rule:9a6a725d'

    ok

rule-actions 命令
^^^^^^^^^^^^^^^^^^^

+-------------------------------------+--------------------+
| rule-actions list [-k [<eventype>]] | List actions       |
+-------------------------------------+--------------------+
| rule-actions show <ActionId>        | Show a rule action |
+-------------------------------------+--------------------+

.. note:: 动作可以由 emqx 内置(称为系统内置动作)，或者由 emqx 插件编写，但不能通过 CLI/API 添加或删除。

rule-actions show
"""""""""""""""""

查询动作::

    ## 查询名为 'inspect' 的动作
    $ ./bin/emqx_ctl rule-actions show 'inspect'

    action(name='inspect', app='emqx_rule_engine', for='$any', types=[], title ='Inspect (debug)', description='Inspect the details of action params for debug purpose')

rule-actions list
"""""""""""""""""

列出符合条件的动作::

    ## 列出当前所有的动作
    $ ./bin/emqx_ctl rule-actions list

    action(name='data_to_rabbit', app='emqx_bridge_rabbit', for='$any', types=[bridge_rabbit], title ='Data bridge to RabbitMQ', description='Store Data to Kafka')
    action(name='data_to_timescaledb', app='emqx_backend_pgsql', for='$any', types=[timescaledb], title ='Data to TimescaleDB', description='Store data to TimescaleDB')
    ...

    ## 列出所有 EventType 类型匹配 'client.connected' 的动作
    ## '$any' 表明此动作可以绑定到到所有类型的事件上。
    $ ./bin/emqx_ctl rule-actions list -k 'client.connected'

    action(name='data_to_cassa', app='emqx_backend_cassa', for='$any', types=[backend_cassa], title ='Data to Cassandra', description='Store data to Cassandra')
    action(name='data_to_dynamo', app='emqx_backend_dynamo', for='$any', types=[backend_dynamo], title ='Data to DynamoDB', description='Store Data to DynamoDB')
    ...


resources 命令
^^^^^^^^^^^^^^^^

+--------------------------------------------------------+-------------------+
| resources create <type> [-c [<config>]] [-d [<descr>]] | Create a resource |
+--------------------------------------------------------+-------------------+
| resources list [-t <ResourceType>]                     | List resources    |
+--------------------------------------------------------+-------------------+
| resources show <ResourceId>                            | Show a resource   |
+--------------------------------------------------------+-------------------+
| resources delete <ResourceId>                          | Delete a resource |
+--------------------------------------------------------+-------------------+

resources create
""""""""""""""""

创建一个新的资源，参数:

- type: 资源类型
- -c config: JSON 格式的配置
- -d descr: 可选，资源的描述

::

    $ ./bin/emqx_ctl resources create 'web_hook' -c '{"url": "http://host-name/chats"}' -d 'forward msgs to host-name/chats'

    Resource resource:a7a38187 created

resources list
""""""""""""""

列出当前所有的资源::

    $ ./bin/emqx_ctl resources list

    resource(id='resource:a7a38187', type='web_hook', config=#{<<"url">> => <<"http://host-name/chats">>}, status=#{is_alive => false}, description='forward msgs to host-name/chats')

resources list by type
""""""""""""""""""""""

列出当前所有的资源::

    $ ./bin/emqx_ctl resources list --type='web_hook'

    resource(id='resource:a7a38187', type='web_hook', config=#{<<"url">> => <<"http://host-name/chats">>}, status=#{is_alive => false}, description='forward msgs to host-name/chats')

resources show
""""""""""""""

查询资源::

    $ ./bin/emqx_ctl resources show 'resource:a7a38187'

    resource(id='resource:a7a38187', type='web_hook', config=#{<<"url">> => <<"http://host-name/chats">>}, status=#{is_alive => false}, description='forward msgs to host-name/chats')

resources delete
""""""""""""""""

删除资源::

    $ ./bin/emqx_ctl resources delete 'resource:a7a38187'

    ok

resource-types 命令
^^^^^^^^^^^^^^^^^^^^^

+----------------------------+-------------------------+
| resource-types list        | List all resource-types |
+----------------------------+-------------------------+
| resource-types show <Type> | Show a resource-type    |
+----------------------------+-------------------------+

.. note:: 资源类型可以由 emqx 内置(称为系统内置资源类型)，或者由 emqx 插件编写，但不能通过 CLI/API 添加或删除。

resource-types list
"""""""""""""""""""

列出当前所有的资源类型::

    ./bin/emqx_ctl resource-types list

    resource_type(name='backend_mongo_rs', provider='emqx_backend_mongo', title ='MongoDB Replica Set Mode', description='MongoDB Replica Set Mode')
    resource_type(name='backend_cassa', provider='emqx_backend_cassa', title ='Cassandra', description='Cassandra Database')
    ...

resource-types show
"""""""""""""""""""

查询资源类型::

    $ ./bin/emqx_ctl resource-types show backend_mysql

    resource_type(name='backend_mysql', provider='emqx_backend_mysql', title ='MySQL', description='MySQL Database')


.. _rule_engine.api:

规则引擎 HTTP API
--------------------

规则 API
^^^^^^^^^

创建规则
"""""""""

API 定义::

  POST api/v3/rules

参数定义:

+------------------+-------------------------------------------+
| rawsql           | String，用于筛选和转换原始数据的 SQL 语句 |
+------------------+-------------------------------------------+
| actions          | JSON Array，动作列表                      |
+------------------+-------------------------------------------+
| - actions.name   | String, 动作名字                          |
+------------------+-------------------------------------------+
| - actions.params | JSON Object, 动作参数                     |
+------------------+-------------------------------------------+
| description      | String，可选，规则描述                    |
+------------------+-------------------------------------------+

API 请求示例::

    GET http://localhost:8080/api/v3/rules

API 请求消息体:

.. code-block:: json

  {
    "rawsql": "select * from \"message.publish\"",
    "actions": [{
        "name": "inspect",
        "params": {
            "a": 1
        }
    }],
    "description": "test-rule"
  }

API 返回数据示例:

.. code-block:: json

  {
    "code": 0,
    "data": {
        "actions": [{
            "name": "inspect",
            "params": {
                "a": 1
            }
        }],
        "description": "test-rule",
        "enabled": true,
        "for": "message.publish",
        "id": "rule:34476883",
        "rawsql": "select * from \"message.publish\""
    }
  }

查询规则
"""""""""

API 定义::

  GET api/v3/rules/:id

API 请求示例::

  GET api/v3/rules/rule:34476883

API 返回数据示例:

.. code-block:: json

  {
    "code": 0,
    "data": {
        "actions": [{
            "name": "inspect",
            "params": {
                "a": 1
            }
        }],
        "description": "test-rule",
        "enabled": true,
        "for": "message.publish",
        "id": "rule:34476883",
        "rawsql": "select * from \"message.publish\""
    }
  }

获取当前规则列表
""""""""""""""""

API 定义::

  GET api/v3/rules

API 返回数据示例:

.. code-block:: json

  {
    "code": 0,
    "data": [{
        "actions": [{
            "name": "inspect",
            "params": {
                "a": 1
            }
        }],
        "description": "test-rule",
        "enabled": true,
        "for": "message.publish",
        "id": "rule:34476883",
        "rawsql": "select * from \"message.publish\""
    }]
  }


删除规则
"""""""""

API 定义::

  DELETE api/v3/rules/:id

请求参数示例::

  DELETE api/v3/rules/rule:34476883

API 返回数据示例:

.. code-block:: json

  {
    "code": 0
  }

动作 API
^^^^^^^^^

获取当前动作列表
""""""""""""""""

API 定义::

  GET api/v3/actions?for=${hook_type}

API 请求示例::

  GET api/v3/actions

API 返回数据示例::

  {
    "code": 0,
    "data": [{
        "app": "emqx_rule_engine",
        "description": "Republish a MQTT message to another topic",
        "for": "message.publish",
        "name": "republish",
        "params": {
            "target_topic": {
                "description": "To which topic the message will be republished",
                "format": "topic",
                "required": true,
                "title": "To Which Topic",
                "type": "string"
            }
        },
        "types": []
    },
    ... ]
  }

API 请求示例::

  GET 'api/v3/actions?for=client.connected'

API 返回数据示例:

.. code-block:: json

  {
    "code": 0,
    "data": [{
        "app": "emqx_rule_engine",
        "description": "Inspect the details of action params for debug purpose",
        "for": "$any",
        "name": "inspect",
        "params": {},
        "types": []
    }]
  }

查询动作
"""""""""

API 定义::

  GET api/v3/actions/:action_name

API 请求示例::

  GET 'api/v3/actions/inspect'

API 返回数据示例:

.. code-block:: json

  {
    "code": 0,
    "data": {
        "app": "emqx_rule_engine",
        "description": "Inspect the details of action params for debug purpose",
        "for": "$any",
        "name": "inspect",
        "params": {},
        "types": []
    }
  }

资源类型 API
^^^^^^^^^^^^^

获取当前资源类型列表
""""""""""""""""""""

API 定义::

  GET api/v3/resource_types

返回数据示例::

  {
    "code": 0,
    "data": [{
        "config": {
            "url": "http://host-name/chats"
        },
        "description": "forward msgs to host-name/chats",
        "id": "resource:a7a38187",
        "type": "web_hook"
    },
    ... ]
  }

查询资源类型
"""""""""""""

API 定义::

  GET api/v3/resource_types/:type

返回数据示例::

  GET api/v3/resource_types/web_hook

.. code-block:: json

  {
    "code": 0,
    "data": {
        "description": "WebHook",
        "name": "web_hook",
        "params": {},
        "provider": "emqx_web_hook"
    }
  }

获取某种类型的资源
""""""""""""""""""

API 定义::

  GET api/v3/resource_types/:type/resources

API 请求示例::

  GET api/v3/resource_types/web_hook/resources

API 返回数据示例:

.. code-block:: json

  {
    "code": 0,
    "data": [{
        "config": {"url":"http://host-name/chats"},
        "description": "forward msgs to host-name/chats",
        "id": "resource:6612f20a",
        "type": "web_hook"
    }]
  }


资源 API
^^^^^^^^^

创建资源
"""""""""

API 定义::

  POST api/v3/resources

API 参数定义:

+-------------+------------------------+
| type        | String, 资源类型       |
+-------------+------------------------+
| config      | JSON Object, 资源配置  |
+-------------+------------------------+
| description | String，可选，规则描述 |
+-------------+------------------------+

API 请求参数示例::

  {
    "type": "web_hook",
    "config": {
        "url": "http://127.0.0.1:9910",
        "headers": {"token":"axfw34y235wrq234t4ersgw4t"},
        "method": "POST"
    },
    "description": "web hook resource-1"
  }

API 返回数据示例:

.. code-block:: json

  {
    "code": 0,
    "data": {
        "config": {
            "headers":{"token":"axfw34y235wrq234t4ersgw4t"},
            "method":"POST",
            "url":"http://127.0.0.1:9910"
        },
        "description": "web hook resource-1",
        "id": "resource:62763e19",
        "type": "web_hook"
    }
  }


获取资源列表
""""""""""""

API 定义::

  GET api/v3/resources

API 返回数据示例::

  {
    "code": 0,
    "data": [{
        "config": {
            "headers":{"token":"axfw34y235wrq234t4ersgw4t"},
            "method":"POST",
            "url":"http://127.0.0.1:9910"
        },
        "description": "web hook resource-1",
        "id": "resource:62763e19",
        "type": "web_hook"
    },
    ... ]
  }


查询资源
"""""""""

API 定义::

  GET api/v3/resources/:resource_id

API 返回数据示例::

  GET 'api/v3/resources/resource:62763e19'

.. code-block:: json

  {
    "code": 0,
    "data": {
        "config": {
            "headers":{"token":"axfw34y235wrq234t4ersgw4t"},
            "method":"POST",
            "url":"http://127.0.0.1:9910"
        },
        "description": "web hook resource-1",
        "id": "resource:62763e19",
        "type": "web_hook"
    }
  }

删除资源
"""""""""

API 定义::

  DELETE api/v3/resources/:resource_id

API 返回数据示例::

  DELETE 'api/v3/resources/resource:62763e19'

.. code-block:: json

  {
    "code": 0
  }


.. _rule_engine_examples:

==============
创建规则举例
==============

.. _rule_engine_examples.cli:

通过 CLI 创建简单规则
-----------------------

.. _rule_engine_examples.cli.inspect:

创建 Inspect 规则
^^^^^^^^^^^^^^^^^^^^^

创建一个测试规则，当有消息发送到 't/a' 主题时，打印消息内容以及动作参数细节。

- 规则的筛选 SQL 语句为: SELECT * FROM "message.publish" WHERE topic = 't/a';
- 动作是: "打印动作参数细节"，需要使用内置动作 'inspect'。

.. code-block:: shell

    $ ./bin/emqx_ctl rules create \
      "SELECT * FROM \"message.publish\" WHERE topic = 't/a'" \
      '[{"name":"inspect", "params": {"a": 1}}]' \
      -d 'Rule for debug'

    Rule rule:803de6db created

上面的 CLI 命令创建了一个 ID 为 'Rule rule:803de6db' 的规则。

参数中前两个为必参数:

- SQL 语句: SELECT * FROM "message.publish" WHERE topic = 't/a'
- 动作列表: [{"name":"inspect", "params": {"a": 1}}]。动作列表是用 JSON Array 格式表示的。name 字段是动作的名字，params 字段是动作的参数。注意 ``inspect`` 动作是不需要绑定资源的。

最后一个可选参数，是规则的描述: 'Rule for debug'。

接下来当发送 "hello" 消息到主题 't/a' 时，上面创建的 "Rule rule:803de6db" 规则匹配成功，然后 "inspect" 动作被触发，将消息和参数内容打印到 emqx 控制台::

    $ tail -f log/erlang.log.1

    (emqx@127.0.0.1)1> [inspect]
        Selected Data: #{client_id => <<"shawn">>,event => 'message.publish',
                         flags => #{dup => false,retain => false},
                         id => <<"5898704A55D6AF4430000083D0002">>,
                         payload => <<"hello">>,
                         peername => <<"127.0.0.1:61770">>,qos => 1,
                         timestamp => 1558587875090,topic => <<"t/a">>,
                         username => undefined}
        Envs: #{event => 'message.publish',
                flags => #{dup => false,retain => false},
                from => <<"shawn">>,
                headers =>
                    #{allow_publish => true,
                      peername => {{127,0,0,1},61770},
                      username => undefined},
                id => <<0,5,137,135,4,165,93,106,244,67,0,0,8,61,0,2>>,
                payload => <<"hello">>,qos => 1,
                timestamp => {1558,587875,89754},
                topic => <<"t/a">>}
        Action Init Params: #{<<"a">> => 1}

- ``Selected Data`` 列出的是消息经过 SQL 筛选、提取后的字段，由于我们用的是 ``select *``，所以这里会列出所有可用的字段。
- ``Envs`` 是动作内部可以使用的环境变量。
- ``Action Init Params`` 是初始化动作的时候，我们传递给动作的参数。

.. _rule_engine_examples.cli.webhook:

创建 WebHook 规则
^^^^^^^^^^^^^^^^^^^^^

创建一个规则，将所有发送自 client_id='Steven' 的消息，转发到地址为 'http://127.0.0.1:9910' 的 Web 服务器:

- 规则的筛选条件为: SELECT username as u, payload FROM "message.publish" where u='Steven';
- 动作是: "转发到地址为 'http://127.0.0.1:9910' 的 Web 服务";
- 资源类型是: web_hook;
- 资源是: "到 url='http://127.0.0.1:9910' 的 WebHook 资源"。

0. 首先我们创建一个简易 Web 服务，这可以使用 ``nc`` 命令实现::

    $ while true; do echo -e "HTTP/1.1 200 OK\n\n $(date)" | nc -l 127.0.0.1 9910; done;

1. 使用 WebHook 类型创建一个资源，并配置资源参数 url:

   1). 列出当前所有可用的资源类型，确保 'web_hook' 资源类型已存在::

    $ ./bin/emqx_ctl resource-types list

    resource_type(name='web_hook', provider='emqx_web_hook', params=#{...}}, on_create={emqx_web_hook_actions,on_resource_create}, description='WebHook Resource')
    ...

   2). 使用类型 'web_hook' 创建一个新的资源，并配置 "url"="http://127.0.0.1:9910"::

    $ ./bin/emqx_ctl resources create \
      'web_hook' \
      -c '{"url": "http://127.0.0.1:9910", "headers": {"token":"axfw34y235wrq234t4ersgw4t"}, "method": "POST"}'

    Resource resource:691c29ba created

   上面的 CLI 命令创建了一个 ID 为 'resource:691c29ba' 的资源，第一个参数是必选参数 - 资源类型(web_hook)。参数表明此资源指向 URL = "http://127.0.0.1:9910" 的 Web 服务，方法为 POST，并且设置了一个 HTTP Header: "token"。

2. 然后创建规则，并选择规则的动作为 'data_to_webserver':

   1). 列出当前所有可用的动作，确保 'data_to_webserver' 动作已存在::

    $ ./bin/emqx_ctl rule-actions list

    action(name='data_to_webserver', app='emqx_web_hook', for='$any', types=[web_hook], params=#{'$resource' => ...}, title ='Data to Web Server', description='Forward Messages to Web Server')
    ...

   2). 创建规则，选择 data_to_webserver 动作，并通过 "$resource" 参数将 resource:691c29ba 资源绑定到动作上::

    $ ./bin/emqx_ctl rules create \
     "SELECT username as u, payload FROM \"message.publish\" where u='Steven'" \
     '[{"name":"data_to_webserver", "params": {"$resource":  "resource:691c29ba"}}]' \
     -d "Forward publish msgs from steven to webserver"

    rule:26d84768

   上面的 CLI 命令与第一个例子里创建 Inspect 规则时类似，区别在于这里需要把刚才创建的资源 'resource:691c29ba' 绑定到 'data_to_webserver' 动作上。这个绑定通过给动作设置一个特殊的参数 '$resource' 完成。'data_to_webserver' 动作的作用是将数据发送到指定的 Web 服务器。

3. 现在我们使用 username "Steven" 发送 "hello" 到任意主题，上面创建的规则就会被触发，Web Server 收到消息并回复 200 OK::

    $ while true; do echo -e "HTTP/1.1 200 OK\n\n $(date)" | nc -l 127.0.0.1 9910; done;

    POST / HTTP/1.1
    content-type: application/json
    content-length: 32
    te:
    host: 127.0.0.1:9910
    connection: keep-alive
    token: axfw34y235wrq234t4ersgw4t

    {"payload":"hello","u":"Steven"}
