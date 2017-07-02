.. _rest:

========
REST API
========

------------
数据管理 API
------------

连接
----

获取指定节点的客户端信息列表::

    method   : GET
    URL      : api/v2/nodes/{node_name}/clients
    请求参数  : curr_page=1&page_size=20
    请求试例  : api/v2/nodes/emqx@127.0.0.1/clients?curr_page=1&page_size=20
    返回数据  :
    {
        "code": 0,
        "result":
        {
            "current_page": 1,
            "page_size": 20,
            "total_num": 1,
            "total_page": 1,
            "objects":
            [
                {
                    "client_id": "C_1492145414740",
                    "username": "undefined",
                    "ipaddress": "127.0.0.1",
                    "port": 49639,
                    "clean_sess": true,
                    "proto_ver": 4,
                    "keepalive": 60,
                    "connected_at": "2017-04-14 12:50:15"
                }
            ]
        }   
    }

获取指定节点的指定客户端的信息::

    method   : GET
    URL      : api/v2/nodes/{node_name}/clients/{clientid}
    请求参数  :
    请求试例  : api/v2/nodes/emqx@127.0.0.1/clients/C_1492145414740
    返回数据  :
    {
        "code": 0,
        "result": {
            "objects": [
                {
                    "client_id": "C_1492145414740",
                    "username": "undefined",
                    "ipaddress": "127.0.0.1",
                    "port": 50953,
                    "clean_sess": true,
                    "proto_ver": 4,
                    "keepalive": 60,
                    "connected_at": "2017-04-14 13:35:15"
                }
            ]
        }
    }

获取集群下指定客户端的信息::

    method   : GET
    URL      : api/v2/clients/{clientid}
    请求参数  : 
    请求试例  : api/v2/clients/C_1492145414740
    返回数据  :
    {
        "code": 0,
        "result": {
            "objects": [
                {
                    "client_id": "C_1492145414740",
                    "username": "undefined",
                    "ipaddress": "127.0.0.1",
                    "port": 50953,
                    "clean_sess": true,
                    "proto_ver": 4,
                    "keepalive": 60,
                    "connected_at": "2017-04-14 13:35:15"
                }
            ]
        }
    }


会话
----

获取指定节点的会话信息列表::

    method   : GET
    URL      : api/v2/nodes/{node_name}/sessions
    请求参数  : curr_page=1&page_size=20
    请求试例  : api/v2/nodes/emqx@127.0.0.1/sessions?curr_page=1&page_size=20
    返回数据  :
    {
        "code": 0,
        "result":
        {
            "current_page": 1,
            "page_size": 20,
            "total_num": 1,
            "total_page": 1,
            "objects":
            [
                {
                    "client_id": "C_1492145414740",
                    "clean_sess": true,
                    "max_inflight": "undefined",
                    "inflight_queue": "undefined",
                    "message_queue": "undefined",
                    "message_dropped": "undefined",
                    "awaiting_rel": "undefined",
                    "awaiting_ack": "undefined",
                    "awaiting_comp": "undefined",
                    "created_at": "2017-04-14 13:35:15"
                }
            ]
        }
    }

获取指定节点的指定客户端的会话信息::

    method   : GET
    URL      : api/v2/nodes/{node_name}/sessions/{clientid}
    请求参数  :
    请求试例  : api/v2/nodes/emqx@127.0.0.1/sessions/C_1492145414740
    返回数据  :
    {
        "code": 0,
        "result":
        {
            "current_page": 1,
            "page_size": 20,
            "total_num": 1,
            "total_page": 1,
            "objects":
            [
                {
                    "client_id": "C_1492145414740",
                    "clean_sess": true,
                    "max_inflight": "undefined",
                    "inflight_queue": "undefined",
                    "message_queue": "undefined",
                    "message_dropped": "undefined",
                    "awaiting_rel": "undefined",
                    "awaiting_ack": "undefined",
                    "awaiting_comp": "undefined",
                    "created_at": "2017-04-14 13:35:15"
                }
            ]
        }
    }

获取指定集群下指定客户端的会话信息::

    method   : GET
    URL      : api/v2/sessions/{clientid}
    请求参数  :
    请求试例  : api/v2/sessions/C_1492145414740
    返回数据  :
    {
        "code": 0,
        "result":
        {
            "current_page": 1,
            "page_size": 20,
            "total_num": 1,
            "total_page": 1,
            "objects":
            [
                {
                    "client_id": "C_1492145414740",
                    "clean_sess": true,
                    "max_inflight": "undefined",
                    "inflight_queue": "undefined",
                    "message_queue": "undefined",
                    "message_dropped": "undefined",
                    "awaiting_rel": "undefined",
                    "awaiting_ack": "undefined",
                    "awaiting_comp": "undefined",
                    "created_at": "2017-04-14 13:35:15"
                }
            ]
        }
    }

订阅
----

获取指定节点的订阅信息列表::

    method   : GET
    URL      : api/v2/nodes/{node_name}/subscriptions
    请求参数  : curr_page=1&page_size=20
    请求试例  : api/v2/nodes/emqx@127.0.0.1/subscriptions?curr_page=1&page_size=20
    返回数据  :
    {
        "code": 0,
        "result":
        {
            "current_page": 1,
            "page_size": 20,
            "total_num": 1,
            "total_page": 1,
            "objects":
            [
                {
                    "client_id": "C_1492145414740",
                    "topic": "$client/C_1492145414740",
                    "qos": 1
                }
            ]
        }
    }

获取指定节点的指定客户端的订阅信息::

    method   : GET
    URL      : api/v2/nodes/{node_name}/subscriptions/{clientid}
    请求参数  :
    请求试例  : api/v2/nodes/emqx@127.0.0.1/subscriptions/C_1492145414740
    返回数据  :
    {
        "code": 0,
        "result":
        {
            "objects":
            [
                {
                    "client_id": "C_1492145414740",
                    "topic": "$client/C_1492145414740",
                    "qos": 1
                }
            ]
        }
    }

获取集群下的指定客户端的订阅信息::

    method   : GET
    URL      : api/v2/subscriptions/{clientid}
    请求参数  :
    请求试例  : api/v2/subscriptions/C_1492145414740
    返回数据  :
    {
        "code": 0,
        "result":
        {
            "objects":
            [
                {
                    "client_id": "C_1492145414740",
                    "topic": "$client/C_1492145414740",
                    "qos": 1
                }
            ]
        }
    }


路由
----

获取集群下路由信息::

    method   : GET
    URL      : api/v2/routers
    请求参数  : curr_page=1&page_size=20
    请求试例  : api/v2/routers?curr_page=1&page_size=20
    返回数据  :
    {
        "code": 0,
        "result":
        {
            "current_page": 1,
            "page_size": 20,
            "total_num": 1,
            "total_page": 1,
            "objects":
            [
                {
                    "topic": "$client/C_1492145414740",
                    "node": "emqx@127.0.0.1"
                }
            ]
        }
    }

获取集群下指定主题的路由信息::

    method   : GET
    URL      : api/v2/routers/{topic}
    请求参数  :
    请求试例  : api/v2/routers/test_topic
    返回数据  :
    {
        "code": 0,
        "result":
        {
            "objects":
            [
                {
                    "topic": "test_topic",
                    "node": "emqx@127.0.0.1"
                }
            ]
        }
    }


插件
----

获取指定节点的插件列表::

    method   : GET
    URL      : api/v2/nodes/{node_name}/plugins
    请求参数  :
    请求试例  : api/v2/nodes/emqx@127.0.0.1/plugins
    返回数据  :
    {
        "code": 0,
        "result":
        [
            {
                "name": "emqx_auth_clientid",
                "version": "2.1.1",
                "description": "EMQ X Authentication with ClientId/Password",
                "active": false
            },
            {
                "name": "emqx_auth_eems",
                "version": "1.0",
                "description": "EMQ X Authentication/ACL with eems",
                "active": false
            },
            {
                "name": "emqx_auth_http",
                "version": "2.1.1",
                "description": "EMQ X Authentication/ACL with HTTP API",
                "active": false
            },
            {
                "name": "emqx_auth_ldap",
                "version": "2.1.1",
                "description": "EMQ X Authentication/ACL with LDAP",
                "active": false
            },
            {
                "name": "emqx_auth_mongo",
                "version": "2.1.1",
                "description": "EMQ X Authentication/ACL with MongoDB",
                "active": false
            },
            {
                "name": "emqx_auth_mysql",
                "version": "2.1.1",
                "description": "EMQ X Authentication/ACL with MySQL",
                "active": false
            },
            {
                "name": "emqx_auth_pgsql",
                "version": "2.1.1",
                "description": "EMQ X Authentication/ACL with PostgreSQL",
                "active": false
            },
            {
                "name": "emqx_auth_redis",
                "version": "2.1.1",
                "description": "EMQ X Authentication/ACL with Redis",
                "active": false
            },
            {
                "name": "emqx_auth_username",
                "version": "2.1.1",
                "description": "EMQ X Authentication with Username/Password",
                "active": false
            },
            {
                "name": "emqx_backend_cassa",
                "version": "2.1.1",
                "description": "EMQ X Cassandra Backend",
                "active": false
            },
            {
                "name": "emqx_backend_mongo",
                "version": "2.1.1",
                "description": "EMQ X Mongodb Backend",
                "active": false
            },
            {
                "name": "emqx_backend_mysql",
                "version": "2.1.0",
                "description": "EMQ X MySQL Backend",
                "active": false
            },
            {
                "name": "emqx_backend_pgsql",
                "version": "2.1.1",
                "description": "EMQ X PostgreSQL Backend",
                "active": false
            },
            {
                "name": "emqx_backend_redis",
                "version": "2.1.1",
                "description": "EMQ X Redis Backend",
                "active": false
            },
            {
                "name": "emqx_bridge_kafka",
                "version": "2.1.1",
                "description": "EMQ X Kafka Bridge",
                "active": false
            },
            {
                "name": "emqx_bridge_rabbit",
                "version": "2.1.1",
                "description": "EMQ X Bridge RabbitMQ",
                "active": false
            },
            {
                "name": "emqx_dashboard",
                "version": "2.1.1",
                "description": "EMQ X Dashboard",
                "active": true
            },
            {
                "name": "emqx_modules",
                "version": "2.1.1",
                "description": "EMQ X Modules",
                "active": true
            },
            {
                "name": "emqx_recon",
                "version": "2.1.1",
                "description": "Recon Plugin",
                "active": true
            },
            {
                "name": "emqx_reloader",
                "version": "2.1.1",
                "description": "Reloader Plugin",
                "active": false
            },
            {
                "name": "emqx_retainer",
                "version": "2.1.1",
                "description": "EMQ X Retainer",
                "active": true
            }
        ]
    }

开启/关闭指定节点的指定插件::

    method   : PUT
    URL      : /api/v2/nodes/{node_name}/plugins/{name}
    请求参数  : {"active": true/false}
    请求试例  : api/v2/nodes/emqx@127.0.0.1/plugins/emqx_recon
    返回数据  :
    {
        "code": 0,
        "result": []
    }

监听器
-----

获取集群下的监听端口信息::

    method   : GET
    URL      : api/v2/monitoring/listeners
    请求参数  :
    请求试例  : api/v2/monitoring/listeners
    返回数据  :
    {
        "code": 0,
        "result":
        {
            "emqx@127.0.0.1":
            [
                {
                    "protocol": "mqtt:tcp",
                    "listen": "127.0.0.1:11883",
                    "acceptors": 16,
                    "max_clients": 102400,
                    "current_clients": 0,
                    "shutdown_count": []
                },
                {
                    "protocol": "mqtt:tcp",
                    "listen": "0.0.0.0:1883",
                    "acceptors": 16,
                    "max_clients": 102400,
                    "current_clients": 0,
                    "shutdown_count": []
                },
                {
                    "protocol": "mqtt:ws",
                    "listen": "8083",
                    "acceptors": 4,
                    "max_clients": 64,
                    "current_clients": 1,
                    "shutdown_count": []
                },
                {
                    "protocol": "mqtt:ssl",
                    "listen": "8883",
                    "acceptors": 16,
                    "max_clients": 102400,
                    "current_clients": 0,
                    "shutdown_count": []
                },
                {
                    "protocol": "mqtt:wss",
                    "listen": "8084",
                    "acceptors": 4,
                    "max_clients": 64,
                    "current_clients": 0,
                    "shutdown_count": []
                }
            ]
        }
    }

获取指定节点的监听端口信息::

    method   : GET
    URL      : api/v2/monitoring/listeners/{node_name}
    请求参数  :
    请求试例  : api/v2/monitoring/listeners/emqx@127.0.0.1
    返回数据  :
    {
        "code": 0,
        "result":
        [
            {
                "protocol": "mqtt:wss",
                "listen": "8084",
                "acceptors": 4,
                "max_clients": 64,
                "current_clients": 0,
                "shutdown_count": []
            },
            {
                "protocol": "mqtt:ssl",
                "listen": "8883",
                "acceptors": 16,
                "max_clients": 102400,
                "current_clients": 0,
                "shutdown_count": []
            },
            {
                "protocol": "mqtt:ws",
                "listen": "8083",
                "acceptors": 4,
                "max_clients": 64,
                "current_clients": 1,
                "shutdown_count": []
            },
            {
                "protocol": "mqtt:tcp",
                "listen": "0.0.0.0:1883",
                "acceptors": 16,
                "max_clients": 102400,
                "current_clients": 0,
                "shutdown_count": []
            },
            {
                "protocol": "mqtt:tcp",
                "listen": "127.0.0.1:11883",
                "acceptors": 16,
                "max_clients": 102400,
                "current_clients": 0,
                "shutdown_count": []
            }
        ]
    }


集群
----

获取指定节点的信息::

获取集群下节点的信息::

获取集群下管理的节点列表::



指标
----

获取集群下节点的指标信息::

获取指定节点的指标信息::

统计
----

获取集群下节点的统计信息::

获取指定节点的统计信息::


发布/订阅
--------

发布消息::


代理订阅::


用户管理 API
------------


登录::

新增用户::

查询某个用户::

查询用户列表::

更新用户::

删除用户::

修改用户密码::
