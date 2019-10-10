
.. _schema_registry:

********
编解码
********

================
EMQ X 编解码介绍
================

Schema Registry 提供了数据编解码能力。

Schema Registry 目前可支持三种格式的编解码：`Avro <https://avro.apache.org>`_，`Protobuf <https://developers.google.com/protocol-buffers/>`_，以及自定义编码。其中 Avro 和 Protobuf 是依赖 Schema 的数据格式，编码后的数据为二进制，解码后为 Map [#f1]_ 格式 。解码后的数据可直接被规则引擎和其他插件使用。用户自定义的 (3rd-party) 编解码服务通过 HTTP 或 TCP 回调的方式，进行更加贴近业务需求的编解码。

.. important:: Schema Registry 为 Avro 和 Protobuf 等内置编码格式维护 Schema 文本，但对于自定义编解码 (3rd-party) 格式，如需要，Schema 文本需由编解码服务自己维护

============
Schema 教程
============

设计原则和教程参见 `Schema Registry 教程 <https://docs.emqx.io/tutorial/v3/cn/rule_engine/schema_register.html>`_

.. _schema_management:

============
Schema 管理
============

Schema 名字
------------

创建 Schema 的时候，需要给定 Schema 的名字。名字的格式为字母数字或者下划线的组合，且第一个字符不为数字。名字中不能包含冒号(":")。

Schema 版本
------------

对于某个名字的 Schema 创建的时候，Schema Registry 会分配相应的版本。版本是为了注明 Schema 之间的兼容性。Schema 的版本管理如下：

- 版本号是针对名字相同的 Schema 的。版本号总是从 "1.0" 开始。

- 每次用同样的名字创建 Schema 的时候，Schema Registry 会检查 Schema 与上一个版本之间的兼容性。如果兼容则次版本号会递增，比如 "1.1"，"1.2"。如果不兼容创建将会失败。

- 如果使用了强制创建参数，主版本号将会递增，比如 "2.0", "3.0"。

- 版本管理只针对有 Avro 或 Protobuf，自定义的编解码没有 Schema，也就没有版本管理。

Schema ID
---------

创建 Schema 成功之后，系统会分配一个 Schema ID。Schema ID 可用在规则引擎的 SQL 语句里。

.. _schema_registry.api:

HTTP API
--------

创建
^^^^^^

API 定义
""""""""""

  POST api/v3/schemas

参数定义
""""""""""

+-------------------------------+-------------------------------------------------------------------------------------------------+
| name                          | 必选。Schema 的名字，字符串类型                                                                 |
+-------------------------------+-------------------------------------------------------------------------------------------------+
| parser_type                   | 必选。编解码类型，可以为 "avro", "protobuf" 或 "3rd-party" 其中之一                             |
+-------------------------------+-------------------------------------------------------------------------------------------------+
| schema                        | 当 Avro 或 Protobuf 类型时必选。Schema 模板，字符串类型。"3rd-party" 类型不需要此字段           |
+-------------------------------+-------------------------------------------------------------------------------------------------+
| parser_addr                   | 当 3rd-party 类型时必选。自定义编解码的服务地址，JSON Object类型。仅用于 "3rd-party" 类型       |
+-------------------------------+-------------------------------------------------------------------------------------------------+
| - parser_addr.url             | 仅当 HTTP 编解码时设置。编解码服务的 HTTP 路径，字符串类型。  例如: http:/127.0.0.1:9001/parser |
+-------------------------------+-------------------------------------------------------------------------------------------------+
| - parser_addr.server          | 仅当 TCP 编解码时设置。编解码服务的 IP 和端口，字符串类型。 例如: 127.0.0.1:9001                |
+-------------------------------+-------------------------------------------------------------------------------------------------+
| - parser_addr.resource_id     | 期望使用现有 Resource 时设置。Resource 的 ID 值，字符串类型。 例如: resource:2be4ef82           |
+-------------------------------+-------------------------------------------------------------------------------------------------+
| parser_opts                   | 当 3rd-party 类型时必选。编解码选项，JSON Object 类型。仅用于 "3rd-party" 类型                  |
+-------------------------------+-------------------------------------------------------------------------------------------------+
| - parser_opts.3rd_party_opts  | 必选。自定义编解码的选项，内容业务相关，字符串类型。                                            |
+-------------------------------+-------------------------------------------------------------------------------------------------+
| - parser_opts.connect_timeout | 可选。自定义编解码的连接超时设置，整数类型，单位秒。                                            |
+-------------------------------+-------------------------------------------------------------------------------------------------+
| - parser_opts.parse_timeout   | 可选。自定义编解码的编解码超时设置，整数类型，单位秒。                                          |
+-------------------------------+-------------------------------------------------------------------------------------------------+
| force                         | 可选。Boolean，可选，是否强制创建。强制创建将不检查与上个版本的兼容性，并跃升主版本号           |
+-------------------------------+-------------------------------------------------------------------------------------------------+
| description                   | 可选。String，可选，规则描述                                                                    |
+-------------------------------+-------------------------------------------------------------------------------------------------+

API 请求示例
"""""""""""""

POST http://localhost:8080/api/v3/schemas

API 请求消息体
"""""""""""""""

.. code-block:: json

  {
    "name":"sensor_notify_avro",
    "parser_type":"avro",
    "description":"Test Avro Schema",
    "schema":"{\"type\": \"record\", \"fields\": [{\"name\": \"name\", \"type\": \"string\"}, {\"name\": \"favorite_number\", \"type\": [\"int\", \"null\"]}, {\"name\": \"favorite_color\", \"type\": [\"string\", \"null\"]}]}"
  }

API 返回数据示例
"""""""""""""""""

.. code-block:: json

  {
    "code":0,
    "data":{
      "id":"sensor_notify_avro:1.0",
      "name":"sensor_notify_avro",
      "version":"1.0",
      "schema":"{\"type\":\"record\",\"fields\":[{\"name\":\"name\",\"type\":\"string\"},{\"name\":\"favorite_number\",\"type\":[\"int\",\"null\"]},{\"name\":\"favorite_color\",\"type\":[\"string\",\"null\"]}]}",
      "parser_type":"avro",
      "parser_addr":null,
      "parser_opts":{},
      "description":"Test Avro Schema"
    }
  }

cURL 示例
"""""""""

创建 Avro Schema::

    ## This appid and secret can be created in emqx dashboard.
    $ APPSECRET='a78ed1495de28:Mjg5MzU2MDY1NTU5MTM4Mjk4Nzg3MjgwOTEwNDExMzY2NDA'

    $ SCHEMA='{"type": "record", "fields": [{"name": "name", "type": "string"}, {"name": "favorite_number", "type": ["int", "null"]}, {"name": "favorite_color", "type": ["string", "null"]}]}'

    $ curl --basic -u $APPSECRET -k 'http://localhost:8080/api/v3/schemas' -d \
    '{"name":"sensor_notify_avro", "parser_type": "avro", "description":"Test Avro Schema", "schema": '$SCHEMA'}'

    {"code":0,"data":{"id":"sensor_notify_avro:1.0","name":"sensor_notify_avro","version":"1.0","schema":"...","parser_type":"avro","parser_addr":null,"parser_opts":{},"description":"Test Avro Schema"}}

创建 Protobuf Schema::

    ## ProtoBuf
    $ APPSECRET='a78ed1495de28:Mjg5MzU2MDY1NTU5MTM4Mjk4Nzg3MjgwOTEwNDExMzY2NDA'
    $ SCHEMA='message Person {
      required string name = 1;
      required int32 id = 2;
      optional string email = 3;

      enum PhoneType {
        MOBILE = 0;
        HOME = 1;
        WORK = 2;
      }

      message PhoneNumber {
        required string number = 1;
        optional PhoneType type = 2 [default = HOME];
      }

      repeated PhoneNumber phones = 4;
    }

    message AddressBook {
      repeated Person people = 1;
    }'

    $ curl --basic -u $APPSECRET -k 'http://localhost:8080/api/v3/schemas' -d \
    '{"name":"sensor_notify_protobuf", "parser_type": "protobuf", "schema": "'$SCHEMA'"}'

    {"code":0,"data":{"id":"sensor_notify_protobuf:1.0","name":"sensor_notify_protobuf","version":"1.0","schema":"...","parser_type":"protobuf","parser_addr":null,"parser_opts":{},"description":""}}

创建第三方编解码::

    ## HTTP
    $ APPSECRET='a78ed1495de28:Mjg5MzU2MDY1NTU5MTM4Mjk4Nzg3MjgwOTEwNDExMzY2NDA'
    $ curl --basic -u $APPSECRET -k 'http://localhost:8080/api/v3/schemas' -d \
    '{"name":"my_http_parser", "parser_type": "3rd-party", "parser_addr": {"url": "http://127.0.0.1:8000/parser"}, "parser_opts": {"3rd_party_opts": "xxxx,xxx", "connect_timeout": 3, "parse_timeout": 5}}'

    {"code":0,"data":{"id":"my_http_parser","name":"my_http_parser","version":"1.0","schema":"...","parser_type":"protobuf","parser_addr":null,"parser_opts":{},"description":""}}

    ## TCP
    $ APPSECRET='a78ed1495de28:Mjg5MzU2MDY1NTU5MTM4Mjk4Nzg3MjgwOTEwNDExMzY2NDA'
    $ curl --basic -u $APPSECRET -k 'http://localhost:8080/api/v3/schemas' -d \
    '{"name":"my_tcp_parser", "parser_type": "3rd-party", "parser_addr": {"server": "127.0.0.1:2291"}, "parser_opts": {"3rd_party_opts": "xxxx,xxx", "connect_timeout": 3, "parse_timeout": 5}}'

    ## or using resource as `parser_addr`:
    $ APPSECRET='a78ed1495de28:Mjg5MzU2MDY1NTU5MTM4Mjk4Nzg3MjgwOTEwNDExMzY2NDA'
    $ curl --basic -u $APPSECRET -k 'http://localhost:8080/api/v3/schemas' -d \
    '{"name":"my_parser", "parser_type": "3rd-party", "parser_addr": {"resource_id": "resource:2be4ef82"}, "parser_opts": {"3rd_party_opts": "xxxx,xxx", "connect_timeout": 3, "parse_timeout": 5}}'

.. important:: 创建第三方编码时，会尝试连接指定地址的服务。如果连接失败，创建将会失败。

查询
^^^^^^

列出全部 Schema::

  GET api/v3/schemas

查询指定 Schema::

  GET api/v3/schemas/${schema_id}

查询某个 Schema 的所有版本::

  GET api/v3/schemas/${name}:*

API 请求示例
"""""""""""""

1. 查询 sensor_notify_avro 的 1.0 版本:

GET http://localhost:8080/api/v3/schemas/sensor_notify_avro:1.0

2. 查询 sensor_notify_avro 的所有版本:

GET http://localhost:8080/api/v3/schemas/sensor_notify_avro:*

API 返回数据示例
"""""""""""""""""

1.

.. code-block:: json

  {
    "code":0,
    "data":[
      {
        "id":"sensor_notify_avro:1.0",
        "name":"sensor_notify_avro",
        "version":"1.0",
        "schema":" ... ",
        "parser_type":"avro",
        "parser_addr":null,
        "parser_opts":{},
        "description":"Schema for notification report from sensors, in avro format"
      }
    ]
  }

2.

.. code-block:: json

  {
    "code":0,
    "data":[
      {
        "id":"sensor_notify_avro:1.0",
        "name":"sensor_notify_avro",
        "version":"1.0",
        "schema":" ... ",
        "parser_type":"avro",
        "parser_addr":null,
        "parser_opts":{},
        "description":"Test Avro Schema"
      },
      {
        "id":"sensor_notify_avro:1.1",
        "name":"sensor_notify_avro",
        "version":"1.1",
        "schema":" ... ",
        "parser_type":"avro",
        "parser_addr":null,
        "parser_opts":{},
        "description":"Test Avro Schema"
      }
    ]
  }

cURL 示例
"""""""""

查询 sensor_notify_avro 的所有版本::

    $ APPSECRET='a78ed1495de28:Mjg5MzU2MDY1NTU5MTM4Mjk4Nzg3MjgwOTEwNDExMzY2NDA'

    $ curl --basic -u $APPSECRET -k 'http://localhost:8080/api/v3/schemas/sensor_notify_avro:*'

    {"code":0,"data":[{"id":"sensor_notify_avro:1.0","name":"sensor_notify_avro","version":"1.0","schema":"...","parser_type":"avro","parser_addr":null,"parser_opts":{},"descr":"Schema for notification report from sensors, in avro format"}]}


删除
^^^^^^

删除指定 Schema::

  DELETE api/v3/schemas/${schema_id}

删除某个 Schema 的所有版本::

  DELETE api/v3/schemas/${name}:*

API 请求示例
"""""""""""""

1. 删除 sensor_notify_avro 的 1.0 版本:

DELETE http://localhost:8080/api/v3/schemas/sensor_notify_avro:1.0

2. 删除 sensor_notify_avro 的所有版本:

DELETE http://localhost:8080/api/v3/schemas/sensor_notify_avro:*

API 返回数据示例
"""""""""""""""""

.. code-block:: json

  {
    "code":0
  }

cURL 示例
"""""""""

删除 sensor_notify_avro 的所有版本::

    $ APPSECRET='a78ed1495de28:Mjg5MzU2MDY1NTU5MTM4Mjk4Nzg3MjgwOTEwNDExMzY2NDA'

    $ curl -XDELETE -v --basic -u $APPSECRET -k 'http://localhost:8080/api/v3/schemas/sensor_notify_avro:*'

    {"code":0}

.. rubric:: Footnotes

.. [#f1] Erlang Map，是规则引擎内部使用的 Key-Value 数据结构. 举例: #{id => 1, name => "Steve"}，定义了一个 id 为 1，name 为 "Steve" 的 Map。
