
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
                }
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

- 从任意 topic 的 JSON 消息体(payload) 中提取 x 字段，并创建别名 x 以便在 WHERE 子句中使用。WHERE 子句限定条件为 x = 1。注意如果 payload 不是合法的 JSON 格式，JSON 解码会失败，导致整个语句匹配失败。下面这个 SQL 语句可以匹配到消息体 {"x": 1}, 但不能匹配到消息体 {"x": 2}:

  3.4.0 以及更老版本::

    SELECT payload as p FROM "message.publish" WHERE p.x = 1

  3.4.1 以及以后版本::

    SELECT json_decode(payload) as p FROM "message.publish" WHERE p.x = 1

- 类似于上面的 SQL 语句，但嵌套地提取消息体中的数据，下面的 SQL 语句可以匹配到 JSON 消息体 {"x": {"y": 1}}:

  3.4.0 以及更老版本::

    SELECT payload as a FROM "message.publish" WHERE a.x.y = 1

  3.4.1 以及以后版本::

    SELECT json_decode(payload) as a FROM "message.publish" WHERE a.x.y = 1

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
    - 3.4.1 版本后，SELECT 子句中，若使用 ``"."`` 符号对 JSON 格式的 payload 进行嵌套选择，必须使用 json_decode() 函数对 payload 进行解码。

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

.. _rule_sql.funcs:

SQL 语句中可用的函数
--------------------

算数函数
^^^^^^^^^^^

+--------+----------+-------------------------+----------+
| 函数名 | 函数作用 | 参数                    | 返回值   |
+--------+----------+-------------------------+----------+
| +      | 加法     | 1. 左操作数 2. 右操作数 | 加和     |
+--------+----------+-------------------------+----------+
| -      | 减法     | 1. 左操作数 2. 右操作数 | 差值     |
+--------+----------+-------------------------+----------+
| *      | 乘法     | 1. 左操作数 2. 右操作数 | 乘积     |
+--------+----------+-------------------------+----------+
| /      | 除法     | 1. 左操作数 2. 右操作数 | 商值     |
+--------+----------+-------------------------+----------+
| div    | 整数除法 | 1. 左操作数 2. 右操作数 | 整数商值 |
+--------+----------+-------------------------+----------+
| mod    | 取模     | 1. 左操作数 2. 右操作数 | 模       |
+--------+----------+-------------------------+----------+


数学函数
^^^^^^^^^^^

+--------+----------------+-----------------------------+--------------+
| 函数名 | 函数作用       | 参数                        | 返回值       |
+--------+----------------+-----------------------------+--------------+
| abs    | 绝对值         | 1. 被操作数                 | 绝对值       |
+--------+----------------+-----------------------------+--------------+
| cos    | 余弦           | 1. 被操作数                 | 余弦值       |
+--------+----------------+-----------------------------+--------------+
| cosh   | 双曲余弦       | 1. 被操作数                 | 双曲余弦值   |
+--------+----------------+-----------------------------+--------------+
| acos   | 反余弦         | 1. 被操作数                 | 反余弦值     |
+--------+----------------+-----------------------------+--------------+
| acosh  | 反双曲余弦     | 1. 被操作数                 | 反双曲余弦值 |
+--------+----------------+-----------------------------+--------------+
| sin    | 正弦           | 1. 被操作数                 | 正弦值       |
+--------+----------------+-----------------------------+--------------+
| sinh   | 双曲正弦       | 1. 被操作数                 | 双曲正弦值   |
+--------+----------------+-----------------------------+--------------+
| asin   | 反正弦         | 1. 被操作数                 | 值           |
+--------+----------------+-----------------------------+--------------+
| asinh  | 反双曲正弦     | 1. 被操作数                 | 反双曲正弦值 |
+--------+----------------+-----------------------------+--------------+
| tan    | 正切           | 1. 被操作数                 | 正切值       |
+--------+----------------+-----------------------------+--------------+
| tanh   | 双曲正切       | 1. 被操作数                 | 双曲正切值   |
+--------+----------------+-----------------------------+--------------+
| atan   | 反正切         | 1. 被操作数                 | 反正切值     |
+--------+----------------+-----------------------------+--------------+
| atanh  | 反双曲正切     | 1. 被操作数                 | 反双曲正切值 |
+--------+----------------+-----------------------------+--------------+
| ceil   | 上取整         | 1. 被操作数                 | 整数值       |
+--------+----------------+-----------------------------+--------------+
| floor  | 下取整         | 1. 被操作数                 | 整数值       |
+--------+----------------+-----------------------------+--------------+
| round  | 四舍五入       | 1. 被操作数                 | 整数值       |
+--------+----------------+-----------------------------+--------------+
| exp    | 幂运算         | 1. 被操作数                 | e 的 x 次幂  |
+--------+----------------+-----------------------------+--------------+
| power  | 指数运算       | 1. 左操作数 x 2. 右操作数 y | x 的 y 次方  |
+--------+----------------+-----------------------------+--------------+
| sqrt   | 平方根运算     | 1. 被操作数                 | 平方根       |
+--------+----------------+-----------------------------+--------------+
| fmod   | 负点数取模函数 | 1. 左操作数 2. 右操作数     | 模           |
+--------+----------------+-----------------------------+--------------+
| log    | 以 e 为底对数  | 1. 被操作数                 | 值           |
+--------+----------------+-----------------------------+--------------+
| log10  | 以 10 为底对数 | 1. 被操作数                 | 值           |
+--------+----------------+-----------------------------+--------------+
| log2   | 以 2 为底对数  | 1. 被操作数                 | 值           |
+--------+----------------+-----------------------------+--------------+

字符串函数
^^^^^^^^^^^

+---------+--------------+-------------------------------------------------------------------------------------------------------+--------------------+
| 函数名  | 函数作用     | 参数                                                                                                  | 返回值             |
+---------+--------------+-------------------------------------------------------------------------------------------------------+--------------------+
| lower   | 转为小写     | 1. 输入字符串                                                                                         | 小写字符串         |
+---------+--------------+-------------------------------------------------------------------------------------------------------+--------------------+
| upper   | 转为大写     | 1. 输入字符串                                                                                         | 大写字符串         |
+---------+--------------+-------------------------------------------------------------------------------------------------------+--------------------+
| trim    | 去掉左右空格 | 1. 输入字符串                                                                                         | 输出字符串         |
+---------+--------------+-------------------------------------------------------------------------------------------------------+--------------------+
| ltrim   | 去掉左空格   | 1. 输入字符串                                                                                         | 输出字符串         |
+---------+--------------+-------------------------------------------------------------------------------------------------------+--------------------+
| rtrim   | 去掉右空格   | 1. 输入字符串                                                                                         | 输出字符串         |
+---------+--------------+-------------------------------------------------------------------------------------------------------+--------------------+
| reverse | 字符串反转   | 1. 输入字符串                                                                                         | 输出字符串         |
+---------+--------------+-------------------------------------------------------------------------------------------------------+--------------------+
| strlen  | 字符串长度   | 1. 输入字符串                                                                                         | 整数值             |
+---------+--------------+-------------------------------------------------------------------------------------------------------+--------------------+
| substr  | 取字符的子串 | 1. 输入字符串 2. 起始位置. 注意: 下标从 1 开始                                                        | 子串               |
+---------+--------------+-------------------------------------------------------------------------------------------------------+--------------------+
| substr  | 取字符的子串 | 1. 输入字符串 2. 起始位置 3. 终止位置. 注意: 下标从 1 开始                                            | 子串               |
+---------+--------------+-------------------------------------------------------------------------------------------------------+--------------------+
| split   | 字符串分割   | 1. 输入字符串 2. 分割符子串                                                                           | 分割后的字符串数组 |
+---------+--------------+-------------------------------------------------------------------------------------------------------+--------------------+
| split   | 字符串分割   | 1. 输入字符串 2. 分割符子串 3. 只查找左边或者右边第一个分隔符, 可选的取值为 'leading' 或者 'trailing' | 分割后的字符串数组 |
+---------+--------------+-------------------------------------------------------------------------------------------------------+--------------------+
| split   | 字符串分割   | 1. 输入字符串 2. 分割符子串 3. 只查找左边或者右边第一个分隔符, 可选的取值为 'leading' 或者 'trailing' | 分割后的字符串数组 |
+---------+--------------+-------------------------------------------------------------------------------------------------------+--------------------+


数组函数
^^^^^^^^^^^

+--------+------------------------------+-----------+-------------+
| 函数名 | 函数作用                     | 参数      | 返回值      |
+--------+------------------------------+-----------+-------------+
| nth    | 取第 n 个元素，下标从 1 开始 | 1. 原数组 | 第 n 个元素 |
+--------+------------------------------+-----------+-------------+

哈希函数
^^^^^^^^^^^

+--------+--------------+---------+-----------+
| 函数名 | 函数作用     | 参数    | 返回值    |
+--------+--------------+---------+-----------+
| md5    | 求 MD5 值    | 1. 数据 | MD5 值    |
+--------+--------------+---------+-----------+
| sha    | 求 SHA 值    | 1. 数据 | SHA 值    |
+--------+--------------+---------+-----------+
| sha256 | 求 SHA256 值 | 1. 数据 | SHA256 值 |
+--------+--------------+---------+-----------+

编解码函数
^^^^^^^^^^^

+---------------+-------------+--------------------------------------------------+---------------+
| 函数名        | 函数作用    | 参数                                             | 返回值        |
+---------------+-------------+--------------------------------------------------+---------------+
| base64_encode | BASE64 编码 | 1. 数据                                          | BASE64 字符串 |
+---------------+-------------+--------------------------------------------------+---------------+
| base64_decode | BASE64 解码 | 1. BASE64 字符串                                 | 数据          |
+---------------+-------------+--------------------------------------------------+---------------+
| json_encode   | JSON 编码   | 1. JSON 字符串                                   | 内部 Map      |
+---------------+-------------+--------------------------------------------------+---------------+
| json_decode   | JSON 解码   | 1. 内部 Map                                      | JSON 字符串   |
+---------------+-------------+--------------------------------------------------+---------------+
| schema_encode | Schema 编码 | 1. Schema ID  2. 内部 Map                        | 数据          |
+---------------+-------------+--------------------------------------------------+---------------+
| schema_encode | Schema 编码 | 1. Schema ID  2. 内部 Map 3. Protobuf Message 名 | 数据          |
+---------------+-------------+--------------------------------------------------+---------------+
| schema_decode | Schema 解码 | 1. Schema ID  2. 数据                            | 内部 Map      |
+---------------+-------------+--------------------------------------------------+---------------+
| schema_decode | Schema 解码 | 1. Schema ID  2. 数据 3. Protobuf Message 名     | 内部 Map      |
+---------------+-------------+--------------------------------------------------+---------------+

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

API 返回数据示例:

.. code-block:: json

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
    }]
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

返回数据示例:

.. code-block:: json

  {
    "code": 0,
    "data": [{
        "config": {
            "url": "http://host-name/chats"
        },
        "description": "forward msgs to host-name/chats",
        "id": "resource:a7a38187",
        "type": "web_hook"
    }]
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

API 返回数据示例:

.. code-block:: json

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
    }]
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

.. _rule_engine_metrics:

=====================================
与规则引擎相关的状态、统计指标和告警
=====================================

规则状态和统计指标
------------------

.. image:: ./_static/images/rule_metrics.png

- 已命中: 规则命中(规则 SQL 匹配成功)的次数，
- 命中速度: 规则命中的速度(次/秒)
- 最大命中速度: 规则命中速度的峰值(次/秒)
- 5分钟平均速度: 5分钟内规则的平均命中速度(次/秒)

动作状态和统计指标
------------------

.. image:: ./_static/images/action_metrics.png

- 成功: 动作执行成功次数
- 失败: 动作执行失败次数

资源状态和告警
---------------

.. image:: ./_static/images/resource_status.png

- 可用: 资源可用
- 不可用: 资源不可用(比如数据库连接断开)

.. _rule_engine_examples:

==============
创建规则举例
==============

通过 CLI 创建数据库和桥接规则
-----------------------------------

:ref:`rule_engine_examples.cli.inspect`

:ref:`rule_engine_examples.cli.webhook`

通过 DashBoard 创建数据库和桥接规则
-----------------------------------

:ref:`rule_engine_examples.dashboard.mysql`

:ref:`rule_engine_examples.dashboard.pgsql`

:ref:`rule_engine_examples.dashboard.cassa`

:ref:`rule_engine_examples.dashboard.mongo`

:ref:`rule_engine_examples.dashboard.dynamodb`

:ref:`rule_engine_examples.dashboard.redis`

:ref:`rule_engine_examples.dashboard.opentsdb`

:ref:`rule_engine_examples.dashboard.timescaledb`

:ref:`rule_engine_examples.dashboard.influxdb`

:ref:`rule_engine_examples.dashboard.webhook`

:ref:`rule_engine_examples.dashboard.kafka`

:ref:`rule_engine_examples.dashboard.pulsar`

:ref:`rule_engine_examples.dashboard.rabbit`

:ref:`rule_engine_examples.dashboard.bridge_mqtt`

:ref:`rule_engine_examples.dashboard.bridge_rpc`
