# REST API

The REST API allows you to query MQTT clients, sessions, subscriptions, and routes. You can also query and monitor the metrics and statistics of the broker.

| Nodes                           | -                                                              |
| ------------------------------- | -------------------------------------------------------------- |
| List all Nodes in the Cluster   | GET api/v2/management/nodes                                    |
| Retrieve a Node’s Info          | GET api/v2/management/nodes/{node_name}                        |
| List all Nodes’statistics in    | GET api/v2/monitoring/nodes the Cluster                        |
| Retrieve a node’s statistics    | GET api/v2/monitoring/nodes/{node_name}                        |
| Clients                         |
| List all Clients on a Node      | GET api/v2/nodes/{node_name}/clients                           |
| Retrieve a Client on a Node     | GET api/v2/nodes/{node_name}/clients/{client_id}               |
| Retrieve a Client in the        | GET api/v2/clients/{client_id} Cluster                         |
| Disconnect a Client             | DELETE api/v2/clients/{clientid}                               |
| Sessions                        |
| List all Sessions on a Node     | GET api/v2/node/{node_name}/sessions                           |
| Retrieve a Session on a Node    | GET api/v2/nodes/{node_name}/sessions/{client_id}              |
| Retrieve a Session in the       | GET api/v2/sessions/{client_id} Cluster                        |
| Subscriptions                   |
| List all Subscriptions of       | GET api/v2/nodes/{node_name}/subscriptions a Node              |
| List Subscriptions of a Client  | GET api/v2/nodes/{node_name}subscriptions/{cliet_id} on a node |
| List Subscriptions of a Client  | GET api/v2/subscriptions/{cliet_id}                            |
| Routes                          |
| List all Routes in the Cluster  | GET api/v2/routes                                              |
| Retrieve a Route in the Cluster | GET api/v2/routes/{topic}                                      |
| Publish/Subscribe/Unsubscribe   |
| Publish MQTT Message            | POST api/v2/mqtt/publish                                       |
| Subscribe                       | POST api/v2/mqtt/subscribe                                     |
| Unsubscribe                     | POST api/v2/mqtt/unsubscribe                                   |
| Plugins                         |
| List all Plugins of a Node      | GET /api/v2/nodes/{node_name}/plugins/                         |
| Start/Stop a Plugin on a node   | PUT /api/v2/nodes/{node_name}/plugins/{plugin_name}            |
| Listeners  | -|
| List all Listeners | GET api/v2/monitoring/listeners  
| List listeners of a Node | GET api/v2/monitoring/listeners/{node_name}  
| Metrics |
| Get Metrics of all Nodes | GET api/v2/monitoring/metrics/  
| Get Metrics of a Node | GET api/v2/monitoring/metrics/{node_name}  
| Statistics  | -
| Get Statistics of all Nodes | GET api/v2/monitoring/stats  
| Get Statistics of a Node | GET api/v2/monitoring/stats/{node_name}

## Base URL

All REST APIs in the documentation have the following base URL:

    http(s)://host:8080/api/v2/

## Basic Authentication

The HTTP requests to the REST API are protected with HTTP Basic authentication, For example:

    curl -v --basic -u \<user>:\<passwd> -k http://localhost:8080/api/v2/nodes/emqx@127.0.0.1/clients

## Nodes

### List all Nodes in the Cluster

Definition:

    GET api/v2/management/nodes

Example Request:

    GET api/v2/management/nodes

Response:

    {
        "code": 0,
        "result":
        [
            {
                "name": "emqx@127.0.0.1",
                "version": "2.1.1",
                "sysdescr": "EMQ X",
                "uptime": "1 hours, 17 minutes, 1 seconds",
                "datetime": "2017-04-14 14 (tel:2017041414):11:38",
                "otp_release": "R19/8.3",
                "node_status": "Running"
            }
        ]
    }

### Retrieve a Node's Info

Definition:

    GET api/v2/management/nodes/{node_name}

Example Request:

    GET api/v2/management/nodes/emqx@127.0.0.1

Response:

    {
        "code": 0,
        "result":
        {
            "version": "2.1.1",
            "sysdescr": "EMQ X",
            "uptime": "1 hours, 17 minutes, 18 seconds",
            "datetime": "2017-04-14 14 (tel:2017041414):11:55",
            "otp_release": "R19/8.3",
            "node_status": "Running"
        }
    }

### List all Nodes'statistics in the Cluster

Definition:

    GET api/v2/monitoring/nodes

Example Request:

    GET api/v2/monitoring/nodes

Response:

    {
        "code": 0,
        "result":
        [
            {
                "name": "emqx@127.0.0.1",
                "otp_release": "R19/8.3",
                "memory_total": "69.19M",
                "memory_used": "49.28M",
                "process_available": 262144,
                "process_used": 303,
                "max_fds": 256,
                "clients": 1,
                "node_status": "Running",
                "load1": "1.93",
                "load5": "1.93",
                "load15": "1.89"
            }
        ]
    }

### Retrieve a node's statistics

Definition:

    GET api/v2/monitoring/nodes/{node_name}

Example Request:

    GET api/v2/monitoring/nodes/emqx@127.0.0.1

Response:

    {
        "code": 0,
        "result":
        {
            "name": "emqx@127.0.0.1",
            "otp_release": "R19/8.3",
            "memory_total": "69.19M",
            "memory_used": "49.24M",
            "process_available": 262144,
            "process_used": 303,
            "max_fds": 256,
            "clients": 1,
            "node_status": "Running",
            "load1": "2.21",
            "load5": "2.00",
            "load15": "1.92"
        }
    }

## Clients

### List all Clients on a Node

Definition:

    GET api/v2/nodes/{node_name}/clients

Request parameter:

    curr_page={page_no}&page_size={page_size}

Example Request:

    GET api/v2/nodes/emqx@127.0.0.1/clients?curr_page=1&page_size=20

Response:

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

### Retrieve a Client on a Node

Definition:

    GET api/v2/nodes/{node_name}/clients/{client_id}

Example Request:

    GET api/v2/nodes/emqx@127.0.0.1/clients/C_1492145414740

Response:

    {
        "code": 0,
        "result":
        {
            "objects":
            [
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

### Retrieve a Client in the Cluster

Definition:

    GET api/v2/clients/{client_id}

Example Request:

    GET api/v2/clients/C_1492145414740

Response:

    {
        "code": 0,
        "result":
        {
            "objects":
            [
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

### Disconnect a Client

Difinition:

    DELETE api/v2/clients/{clientid}

Expample Requst:

    DELETE api/v2/clients/C_1492145414740

Response:

    {
        "code":0,
        "result":[]
    }

## Sessions

### List all Sessions on a Node

Definition:

    GET api/v2/node/{node_name}/sessions

Request parameters:

    curr_page={page_no}&page_size={page_size}

Example Request:

    GET api/v2/nodes/emqx@127.0.0.1/sessions?curr_page=1&page_size=20

Response:

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

### Retrieve a Session on a Node

Definition:

    GET api/v2/nodes/{node_name}/sessions/{client_id}

Example Request:

    GET api/v2/nodes/emqx@127.0.0.1/sessions/C_1492145414740

Response:

    {
        "code": 0,
        "result":
        {
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

### Retrieve Sessions of a client in the Cluster

Definition:

    GET api/v2/sessions/{client_id}

Example Request:

    GET api/v2/sessions/C_1492145414740

Response:

    {
        "code": 0,
        "result":
        {
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

## Subscriptions

### List all Subscriptions of a Node

Definition:

    GET api/v2/nodes/{node_name}/subscriptions

Request parameters:

    curr_page={page_no}&page_size={page_size}

Example Request:

    GET api/v2/nodes/emqx@127.0.0.1/subscriptions?curr_page=1&page_size=20

Response:

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

### List Subscriptions of a client on a node

Definition:

    GET api/v2/nodes/{node_name}/subscriptions/{clientid}

Example Request:

    GET api/v2/nodes/emqx@127.0.0.1/subscriptions/C_1492145414740

Response:

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

### List Subscriptions of a Client

Definition:

    GET api/v2/subscriptions/{cliet_id}

Example Request:

    GET api/v2/subscriptions/C_1492145414740

Response:

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

## Routes

### List all Routes in the Cluster

Definition:

    GET api/v2/routes

Request parameters:

    curr_page={page_no}&page_size={page_size}

Example request:

    GET api/v2/routes

Response:

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

### Retrieve Route Information of a Topic in the Cluster

Definition:

    GET api/v2/routes/{topic}

Example Request:

    GET api/v2/routes/test_topic

Response:

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

## Publish/Subscribe/Unsubscribe

### Publish MQTT Message

Definition:

    POST api/v2/mqtt/publish

Request parameters:

    {
        "topic"         : "test",
        "payload"       : "hello",
        "qos"           : 1,
        "retain"        : false,
        "client_id"     : "C_1492145414740"
    }

::: tip Tip
"topic" is mandatory and other parameters are optional. by default "payload":"", "qos":0, "retain":false, "client_id":"http".
:::

Example request:

    POST api/v2/mqtt/publish

Response:

    {
        "code": 0,
        "result": []
    }

### Subscribe

Definition:

    POST api/v2/mqtt/subscribe

Request parameters:

    {
        "topic"         : "test",
        "qos"           : 1,
        "client_id"     : "C_1492145414740"
    }

Example request:

    POST api/v2/mqtt/subscribe

Response:

    {
        "code"  : 0,
        "result": []
    }

### Unsubscribe

Definition:

    POST api/v2/mqtt/unsubscribe

Request parameters:

    {
        "topic"    : "test",
        "client_id": "C_1492145414740"
    }

Example request:

    POST api/v2/mqtt/unsubscribe

Response:

    {
        "code": 0,
        "result": []
    }

## Plugins

### List all Plugins of a Node

Definition:

    GET /api/v2/nodes/{node_name}/plugins/

    Example request::

     GET api/v2/nodes/emqx@127.0.0.1/plugins

Response:

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

### Start/Stop a Plugin on a node

Definition:

    PUT /api/v2/nodes/{node_name}/plugins/{plugin_name}

Request parameters:

    {
        "active": true/false,
    }

Example request:

    PUT api/v2/nodes/emqx@127.0.0.1/plugins/emqx_recon

Response:

    {
        "code": 0,
        "result": []
    }

## Listeners

### List all Listeners

Definition:

    GET api/v2/monitoring/listeners

Response:

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

### List listeners of a Node

Definition:

    GET api/v2/monitoring/listeners/{node_name}

Example Request:

    GET api/v2/monitoring/listeners/emqx@127.0.0.1

Response:

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

## Metrics

### Get Metrics of all Nodes

Definition:

    GET api/v2/monitoring/metrics/

Response:

    {
        "code": 0,
        "result": {
            "emqx@127.0.0.1":
            {
                "packets/disconnect":0,
                "messages/dropped":0,
                "messages/qos2/received":0,
                "packets/suback":0,
                "packets/pubcomp/received":0,
                "packets/unsuback":0,
                "packets/pingresp":0,
                "packets/puback/missed":0,
                "packets/pingreq":0,
                "messages/retained":3,
                "packets/sent":0,
                "messages/qos2/dropped":0,
                "packets/unsubscribe":0,
                "packets/pubrec/missed":0,
                "packets/connack":0,
                "messages/received":0,
                "packets/pubrec/sent":0,
                "packets/publish/received":0,
                "packets/pubcomp/sent":0,
                "bytes/received":0,
                "packets/connect":0,
                "packets/puback/received":0,
                "messages/sent":0,
                "packets/publish/sent":0,
                "bytes/sent":0,
                "packets/pubrel/missed":0,
                "packets/puback/sent":0,
                "messages/qos0/received":0,
                "packets/subscribe":0,
                "packets/pubrel/sent":0,
                "messages/forward":0,
                "messages/qos2/sent":0,
                "packets/received":0,
                "packets/pubrel/received":0,
                "messages/qos1/received":0,
                "messages/qos1/sent":0,
                "packets/pubrec/received":0,
                "packets/pubcomp/missed":0,
                "messages/qos0/sent":0
            }
        }
    }

### Get Metrics of a Node

Definition:

    GET api/v2/monitoring/metrics/{node_name}

Example Request:

    GET api/v2/monitoring/metrics/emqx@127.0.0.1

Response:

    {
        "code": 0,
        "result": {
            "packets/disconnect":0,
            "messages/dropped":0,
            "messages/qos2/received":0,
            "packets/suback":0,
            "packets/pubcomp/received":0,
            "packets/unsuback":0,
            "packets/pingresp":0,
            "packets/puback/missed":0,
            "packets/pingreq":0,
            "messages/retained":3,
            "packets/sent":0,
            "messages/qos2/dropped":0,
            "packets/unsubscribe":0,
            "packets/pubrec/missed":0,
            "packets/connack":0,
            "messages/received":0,
            "packets/pubrec/sent":0,
            "packets/publish/received":0,
            "packets/pubcomp/sent":0,
            "bytes/received":0,
            "packets/connect":0,
            "packets/puback/received":0,
            "messages/sent":0,
            "packets/publish/sent":0,
            "bytes/sent":0,
            "packets/pubrel/missed":0,
            "packets/puback/sent":0,
            "messages/qos0/received":0,
            "packets/subscribe":0,
            "packets/pubrel/sent":0,
            "messages/forward":0,
            "messages/qos2/sent":0,
            "packets/received":0,
            "packets/pubrel/received":0,
            "messages/qos1/received":0,
            "messages/qos1/sent":0,
            "packets/pubrec/received":0,
            "packets/pubcomp/missed":0,
            "messages/qos0/sent":0
        }
    }

## Statistics

### Get Statistics of all Nodes

Definition:

    GET api/v2/monitoring/stats

Example Request:

    GET api/v2/monitoring/stats

Response:

    {
        "code": 0,
        "result": {
            "emqx@127.0.0.1":
            {
                "clients/count":0,
                "clients/max":0,
                "retained/count":0,
                "retained/max":0,
                "routes/count":0,
                "routes/max":0,
                "sessions/count":0,
                "sessions/max":0,
                "subscribers/count":0,
                "subscribers/max":0,
                "subscriptions/count":0,
                "subscriptions/max":0,
                "topics/count":0,
                "topics/max":0
            }
        }
    }

### Get Statistics of a Node

Definition:

    GET api/v2/monitoring/stats/{node_name}

Example Request:

    GET api/v2/monitoring/stats/emqx@127.0.0.1

Response:

    {
        "code": 0,
        "result": {
            "clients/count":0,
            "clients/max":0,
            "retained/count":0,
            "retained/max":0,
            "routes/count":0,
            "routes/max":0,
            "sessions/count":0,
            "sessions/max":0,
            "subscribers/count":0,
            "subscribers/max":0,
            "subscriptions/count":0,
            "subscriptions/max":0,
            "topics/count":0,
            "topics/max":0
        }
    }

## Error Code

| Code | Comment                         |
| ---- | ------------------------------- |
| 0    | Success                         |
| 101  | badrpc                          |
| 102  | Unknown error                   |
| 103  | Username or password error      |
| 104  | empty username or password      |
| 105  | user does not exist             |
| 106  | admin can not be deleted        |
| 107  | missing request parameter       |
| 108  | request parameter type error    |
| 109  | request parameter is not a json |
| 110  | plugin has been loaded          |
| 111  | plugin has been unloaded        |
| 112  | user is not online              |
