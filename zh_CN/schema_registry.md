# 编解码

## EMQ X 编解码介绍

Schema Registry 提供了数据编解码能力。

Schema Registry 目前可支持三种格式的编解码： [ Avro ](https://avro.apache.org) ， [ Protobuf ](https://developers.google.com/protocol-buffers/) ，以及自定义编码。其中 Avro 和 Protobuf 是依赖 Schema 的数据格式，编码后的数据为二进制，解码后为 Map 1 格式 。解码后的数据可直接被规则引擎和其他插件使用。用户自定义的 (3rd-party) 编解码服务通过 HTTP 或 TCP 回调的方式，进行更加贴近业务需求的编解码。

::: warning Important
Schema Registry 为 Avro 和 Protobuf 等内置编码格式维护 Schema 文本，但对于自定义编解码 (3rd-party) 格式，如需要，Schema 文本需由编解码服务自己维护
:::

## Schema 教程

设计原则和教程参见 [ Schema Registry 教程 ](https://docs.emqx.io/tutorial/v3/cn/rule_engine/schema_register.html)

## Schema 管理

### Schema 名字

创建 Schema 的时候，需要给定 Schema 的名字。名字是 Schema 的唯一标识，不可重复。

Schema 名字的为如下格式的字符串:

- 首字符为字母 `[A-Za-z_]`
- 其余字符为字母数字或者下划线 `[A-Za-z0-9_]`

### HTTP API

#### 创建

#### API 定义

> POST api/v3/schemas

#### 参数定义

| name                           | 必选。Schema 的名字，字符串类型                                                                                                 |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| parser_type                    | 必选。编解码类型，可以为 "avro", "protobuf" 或 "3rd-party" 其中之一                                                             |
| schema                         | 当 Avro 或 Protobuf 类型时必选。Schema 模板，字符串类型。"3rd-party" 类型不需要此字段                                           |
| parser_addr                    | 当 3rd-party 类型时必选。自定义编解码的服务地址，JSON Object 类型。仅用于 "3rd-party" 类型                                      |
| \* parser_addr.url             | 仅当 HTTP 编解码时设置。编解码服务的 HTTP 路径，字符串类型。 例如: [ http:/127.0.0.1:9001/parser ](http:/127.0.0.1:9001/parser) |
| \* parser_addr.server          | 仅当 TCP 编解码时设置。编解码服务的 IP 和端口，字符串类型。 例如: 127.0.0.1:9001                                                |
| \* parser_addr.resource_id     | 期望使用现有 Resource 时设置。Resource 的 ID 值，字符串类型。 例如: [ resource:2be4ef82 ](resource:2be4ef82)                    |
| parser_opts                    | 当 3rd-party 类型时必选。编解码选项，JSON Object 类型。仅用于 "3rd-party" 类型                                                  |
| \* parser_opts.3rd_party_opts  | 必选。自定义编解码的选项，内容业务相关，字符串类型。                                                                            |
| \* parser_opts.connect_timeout | 可选。自定义编解码的连接超时设置，整数类型，单位秒。                                                                            |
| \* parser_opts.parse_timeout   | 可选。自定义编解码的编解码超时设置，整数类型，单位秒。                                                                          |
| description                    | 可选。String，可选，规则描述                                                                                                    |

#### API 请求示例

POST [ http://localhost:8080/api/v3/schemas ](http://localhost:8080/api/v3/schemas)

#### API 请求消息体

    {
      "name":"sensor_notify_avro",
      "parser_type":"avro",
      "description":"Test Avro Schema",
      "schema":"{\"type\": \"record\", \"fields\": [{\"name\": \"name\", \"type\": \"string\"}, {\"name\": \"favorite_number\", \"type\": [\"int\", \"null\"]}, {\"name\": \"favorite_color\", \"type\": [\"string\", \"null\"]}]}"
    }

#### API 返回数据示例

    {
      "code":0,
      "data":{
        "name":"sensor_notify_avro",
        "schema":"{\"type\":\"record\",\"fields\":[{\"name\":\"name\",\"type\":\"string\"},{\"name\":\"favorite_number\",\"type\":[\"int\",\"null\"]},{\"name\":\"favorite_color\",\"type\":[\"string\",\"null\"]}]}",
        "parser_type":"avro",
        "parser_addr":null,
        "parser_opts":{},
        "description":"Test Avro Schema"
      }
    }

#### cURL 示例

创建 Avro Schema:

    ## This appid and secret can be created in emqx dashboard.
    $ APPSECRET='a78ed1495de28:Mjg5MzU2MDY1NTU5MTM4Mjk4Nzg3MjgwOTEwNDExMzY2NDA'

    $ SCHEMA='{"type": "record", "fields": [{"name": "name", "type": "string"}, {"name": "favorite_number", "type": ["int", "null"]}, {"name": "favorite_color", "type": ["string", "null"]}]}'

    $ curl --basic -u $APPSECRET -k 'http://localhost:8080/api/v3/schemas' -d \
    '{"name":"sensor_notify_avro", "parser_type": "avro", "description":"Test Avro Schema", "schema": '$SCHEMA'}'

    {"code":0,"data":{"name":"sensor_notify_avro","schema":"...","parser_type":"avro","parser_addr":null,"parser_opts":{},"description":"Test Avro Schema"}}

创建 Protobuf Schema:

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

    {"code":0,"data":{"name":"sensor_notify_protobuf","schema":"...","parser_type":"protobuf","parser_addr":null,"parser_opts":{},"description":""}}

创建第三方编解码:

    ## HTTP
    $ APPSECRET='a78ed1495de28:Mjg5MzU2MDY1NTU5MTM4Mjk4Nzg3MjgwOTEwNDExMzY2NDA'
    $ curl --basic -u $APPSECRET -k 'http://localhost:8080/api/v3/schemas' -d \
    '{"name":"my_http_parser", "parser_type": "3rd-party", "parser_addr": {"url": "http://127.0.0.1:8000/parser"}, "parser_opts": {"3rd_party_opts": "xxxx,xxx", "connect_timeout": 3, "parse_timeout": 5}}'

    {"code":0,"data":{"name":"my_http_parser","schema":"...","parser_type":"protobuf","parser_addr":null,"parser_opts":{},"description":""}}

    ## TCP
    $ APPSECRET='a78ed1495de28:Mjg5MzU2MDY1NTU5MTM4Mjk4Nzg3MjgwOTEwNDExMzY2NDA'
    $ curl --basic -u $APPSECRET -k 'http://localhost:8080/api/v3/schemas' -d \
    '{"name":"my_tcp_parser", "parser_type": "3rd-party", "parser_addr": {"server": "127.0.0.1:2291"}, "parser_opts": {"3rd_party_opts": "xxxx,xxx", "connect_timeout": 3, "parse_timeout": 5}}'

    ## or using resource as `parser_addr`:
    $ APPSECRET='a78ed1495de28:Mjg5MzU2MDY1NTU5MTM4Mjk4Nzg3MjgwOTEwNDExMzY2NDA'
    $ curl --basic -u $APPSECRET -k 'http://localhost:8080/api/v3/schemas' -d \
    '{"name":"my_parser", "parser_type": "3rd-party", "parser_addr": {"resource_id": "resource:2be4ef82"}, "parser_opts": {"3rd_party_opts": "xxxx,xxx", "connect_timeout": 3, "parse_timeout": 5}}'

::: warning Important
创建第三方编码时，会尝试连接指定地址的服务。如果连接失败，创建将会失败。
:::

#### 查询

列出全部 Schema:

    GET api/v3/schemas

查询指定 Schema:

    GET api/v3/schemas/${schema_id}

#### API 请求示例

查询 sensor_notify_avro:

GET [ http://localhost:8080/api/v3/schemas/sensor_notify_avro ](http://localhost:8080/api/v3/schemas/sensor_notify_avro)

#### API 返回数据示例

    {
      "code":0,
      "data":[
        {
          "name":"sensor_notify_avro",
          "schema":" ... ",
          "parser_type":"avro",
          "parser_addr":null,
          "parser_opts":{},
          "description":"Schema for notification report from sensors, in avro format"
        }
      ]
    }

#### cURL 示例

查询 sensor_notify_avro :

    $ APPSECRET='a78ed1495de28:Mjg5MzU2MDY1NTU5MTM4Mjk4Nzg3MjgwOTEwNDExMzY2NDA'

    $ curl --basic -u $APPSECRET -k 'http://localhost:8080/api/v3/schemas/sensor_notify_avro'

    {"code":0,"data":{"name":"sensor_notify_avro","schema":"...","parser_type":"avro","parser_addr":null,"parser_opts":{},"descr":"Schema for notification report from sensors, in avro format"}}

#### 删除

删除指定 Schema:

    DELETE api/v3/schemas/${schema_id}

#### API 请求示例

删除 sensor_notify_avro:

DELETE [ http://localhost:8080/api/v3/schemas/sensor_notify_avro ](http://localhost:8080/api/v3/schemas/sensor_notify_avro)

#### API 返回数据示例

    {
      "code":0
    }

#### cURL 示例

删除 sensor_notify_avro :

    $ APPSECRET='a78ed1495de28:Mjg5MzU2MDY1NTU5MTM4Mjk4Nzg3MjgwOTEwNDExMzY2NDA'

    $ curl -XDELETE -v --basic -u $APPSECRET -k 'http://localhost:8080/api/v3/schemas/sensor_notify_avro'

    {"code":0}

**Footnotes**

1. Erlang Map，是规则引擎内部使用的 Key-Value 数据结构. 举例: #{id => 1, name => "Steve"}，定义了一个 id 为 1，name 为 "Steve" 的 Map。 ↩︎
