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

获取指定节点的指定会话的信息::

获取集群下指定会话的信息::

订阅
----

获取指定节点的订阅信息列表::

获取指定节点的指定客户端的订阅信息::

获取集群下的指定客户端的订阅信息::


路由
----

获取集群下路由信息::

获取集群下指定主题的路由信息::


插件
----

获取指定节点的插件列表::

开启关闭指定节点的指定插件::


监听器
-----

获取集群下的监听端口信息::

获取指定节点的监听端口信息::


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


------------
用户管理 API
------------


登录::

新增用户::

查询某个用户::

查询用户列表::

更新用户::

删除用户::

修改用户密码::
