
.. _backends:

========
数据存储
========

.. _backend_rule:

------------
数据存储规则
------------

+------------------------+----------------------------------+
| action                 | 说明                             |
+========================+==================================+
| on_client_connected    | 存储客户端在线状态               |
+------------------------+----------------------------------+
| on_subscribe_lookup    | 订阅主题                         |
+------------------------+----------------------------------+
| on_client_disconnected | 存储客户端离线状态               |
+------------------------+----------------------------------+
| on_message_fetch       | 获取离线消息                     |
+------------------------+----------------------------------+
| on_retain_lookup       | 获取retain消息                   |
+------------------------+----------------------------------+
| on_message_publish     | 存储发布消息                     |
+------------------------+----------------------------------+
| on_message_retain      | 存储retain消息                   |
+------------------------+----------------------------------+
| on_retain_delete       | 删除retain消息                   |
+------------------------+----------------------------------+
| on_message_acked       | 存储ACK消息                      |
+------------------------+----------------------------------+

.. _redis_backend:

-------------
Redis数据存储
-------------

配置Redis存储插件
-----------------

配置文件etc/plugins/emq_backend_redis.conf:

.. code-block:: properties

    ## Redis Server
    backend.redis.pool1.server = 127.0.0.1:6379

    ## Redis Pool Size 
    backend.redis.pool1.pool_size = 8

    ## Redis database 
    backend.redis.pool1.database = 1

    ## Redis subscribe channel
    backend.redis.pool1.channel = mqtt_channel

    ## Client Connected Record Rule
    backend.redis.rule.1 = {"action": "on_client_connected",   "pool": "pool1"}

    ## Subscribe Lookup Record Rule
    backend.redis.rule.2 = {"action": "on_subscribe_lookup",   "pool": "pool1"}

    ## Client DisConnected Record Rule
    backend.redis.rule.3 = {"action": "on_client_disconnected", "pool": "pool1"}

    ## Lookup Unread Message Rule
    backend.redis.rule.4 = {"action": "on_message_fetch",      "filter": "#", "pool": "pool1"}

    ## Lookup Retain Message Rule
    backend.redis.rule.5 = {"action": "on_retain_lookup",      "filter": "#", "pool": "pool1"}

    ## Store Publish Message Rule, QOS > 0
    backend.redis.rule.6 = {"action": "on_message_publish",    "filter": "#", "pool": "pool1"}

    ## Store Retain Message Rule
    backend.redis.rule.7 = {"action": "on_message_retain",     "filter": "#", "pool": "pool1"}

    ## Delete Retain Message Rule
    backend.redis.rule.8 = {"action": "on_retain_delete",      "filter": "#", "pool": "pool1"}

    ## Store Ack Rule
    backend.redis.rule.9 = {"action": "on_message_acked",      "filter": "#", "pool": "pool1"}

加载Redis存储插件
-----------------

.. code-block::

    ./bin/emqctl plugins load emq_backend_redis

mqtt:state - 设备在线状态Hash
-----------------------------

*mqtt:state* Hash存储设备在线状态:

.. code-block::

    hmset
    key = mqtt:state:${clientid} 
    value = {state:int, online_at:timestamp, offline_at:timestamp}

    hset
    key = mqtt:client:${node}
    field = ${clientid}
    value = ${ts}

查询设备在线状态::

    HGETALL "mqtt:state:${clientId}"

mqtt:retain - Retain消息
------------------------

.. code-block::

    hmset
    key = mqtt:retain:${topic}
    value = {id: string, from: string, qos: int, topic: string, retain: int, payload: string, ts: timestamp}

查询retain消息::

    HGETALL "mqtt:retain:${topic}"

mqtt:message - 消息存储
-----------------------

*mqtt:state* Hash存储MQTT消息:

.. code-block::

    hmset
    key = mqtt:message:${msgid}
    value = {id: string, from: string, qos: int, topic: string, retain: int, payload: string, ts: timestamp}

    zadd
    key = mqtt:message:${topic}
    field = 1
    value = ${msgid}

    rpush
    key = mqtt:message:${clientid}
    value = ${msgid}

查询某个客户端发布的消息:

.. code-block::

    LRANGE mqtt:message:${clientid} 0 -1

mqtt:acked - 消息确认
---------------------

*mqtt:acked* SET存储客户端消息确认:

.. code-block::

    set
    key = mqtt:acked:${clientid}:${topic}
    value = ${msgid}

mqtt:subscription - 订阅关系
----------------------------

*mqtt:subscription* Hash存储订阅关系:

.. code-block::

    hset
    key = mqtt:subscription:${clientid}
    field = ${topic}
    value = ${qos}

例如为ClientId为"test"的客户端订阅主题topic0, topic1, topic2:

.. code-block::

    HSET "mqtt:subscription:test" "topic0" 0
    HSET "mqtt:subscription:test" "topic1" 1
    HSET "mqtt:subscription:test" "topic2" 2

SUB/UNSUB 事件发布
------------------

当客户端订阅/取消订阅主题时，EMQ向Redis发布事件消息:

.. code-block::

    PUBLISH
    channel = "mqtt_channel"
    message = {type: string , topic: string, clientid: string, qos: int} 
    \*type: [subscribe/unsubscribe]

例如ClientId为test客户端订阅主题topic0::

    PUBLISH "mqtt_channel" "{\"type\": \"subscribe\", \"topic\": \"topic0\", \"clientid\": \"test\", \"qos\": \"0\"}"

例如ClientId为test客户端取消订阅主题::

    PUBLISH "mqtt_channel" "{\"type\": \"unsubscribe\", \"topic\": \"test_topic0\", \"clientid\": \"test\"}"

.. _mysql_backend:

-------------
MySQL数据存储
-------------

配置MySQL存储插件
-----------------

配置文件 etc/plugins/emq_backend_mysql.conf:

.. code-block:: properties

    ## Mysql Server
    backend.mysql.pool1.server = 127.0.0.1:3306

    ## Mysql Pool Size
    backend.mysql.pool1.pool_size = 8

    ## Mysql Username
    backend.mysql.pool1.user = root

    ## Mysql Password
    backend.mysql.pool1.password = public

    ## Mysql Database
    backend.mysql.pool1.database = mqtt

    ## Client Connected Record Rule
    backend.mysql.rule.1 = {"action": "on_client_connected",   "pool": "pool1"}

    ## Subscribe Lookup Record Rule
    backend.mysql.rule.2 = {"action": "on_subscribe_lookup",   "pool": "pool1"}

    ## Client DisConnected Record Rule
    backend.mysql.rule.3 = {"action": "on_client_disconnected", "pool": "pool1"}

    ## Lookup Unread Message Rule
    backend.mysql.rule.4 = {"action": "on_message_fetch",      "filter": "#", "pool": "pool1"}

    ## Lookup Retain Message Rule
    backend.mysql.rule.5 = {"action": "on_retain_lookup",      "filter": "#", "pool": "pool1"}

    ## Store Publish Message Rule, QOS > 0
    backend.mysql.rule.6 = {"action": "on_message_publish",    "filter": "#", "pool": "pool1"}

    ## Store Retain Message Rule
    backend.mysql.rule.7 = {"action": "on_message_retain",     "filter": "#", "pool": "pool1"}

    ## Delete Retain Message Rule
    backend.mysql.rule.8 = {"action": "on_retain_delete",      "filter": "#", "pool": "pool1"}

    ## Store Ack Rule
    backend.mysql.rule.9 = {"action": "on_message_acked",      "filter": "#", "pool": "pool1"}

启用MySQL存储插件
-----------------

.. code-block:: bash

    ./bin/emqctl plugins load emq_backend_mysql

.. code-block:: sql

导入MySQL表结构
---------------

.. NOTE:: 数据库名称可自定义

.. code-block:: bash

    create database mqtt;

    mysql -u root -p mqtt < etc/sql/emq_backend_mysql.sql

MySQL 设备在线状态表(State Table)
---------------------------------

*mqtt_state* 表存储设备在线状态:

.. code-block:: sql

    DROP TABLE IF EXISTS `mqtt_state`;
    CREATE TABLE `mqtt_state` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `clientid` varchar(64) DEFAULT NULL,
      `state` varchar(3) DEFAULT NULL,
      `node` varchar(100) DEFAULT NULL,
      `online_at` datetime DEFAULT NULL,
      `offline_at` datetime DEFAULT NULL,
      `created` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
      PRIMARY KEY (`id`),
      KEY `mqtt_state_idx` (`clientid`),
      UNIQUE KEY `mqtt_state_key` (`clientid`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

查询设备在线状态::

    select * from mqtt_state where clientid = "test";

MySQL 设备订阅主题表(Subscription Table)
----------------------------------------

.. code-block:: sql

    DROP TABLE IF EXISTS `mqtt_subscription`;
    CREATE TABLE `mqtt_subscription` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `clientid` varchar(64) DEFAULT NULL,
      `topic` varchar(256) DEFAULT NULL,
      `qos` int(3) DEFAULT NULL,
      `created` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
      PRIMARY KEY (`id`),
      KEY `mqtt_subscription_idx` (`clientid`,`topic`(255),`qos`),
      UNIQUE KEY `mqtt_subscription_key` (`clientid`,`topic`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

创建订阅示例::

    clientId分别订阅主题topic0, topic1, topic2:

    insert into mqtt_subscription(clientid, topic, qos) values("clientId", "topic0", 0);
    insert into mqtt_subscription(clientid, topic, qos) values("clientId", "topic1", 1);
    insert into mqtt_subscription(clientid, topic, qos) values("clientId", "topic2", 2);

MySQL MQTT消息表(Message Table)
-------------------------------

.. code-block:: sql
    
    DROP TABLE IF EXISTS `mqtt_message`;
    CREATE TABLE `mqtt_message` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `msgid` varchar(100) DEFAULT NULL,
      `topic` varchar(1024) NOT NULL,
      `sender` varchar(1024) DEFAULT NULL,
      `node` varchar(60) DEFAULT NULL,
      `qos` int(11) NOT NULL DEFAULT '0',
      `retain` tinyint(2) DEFAULT NULL,
      `payload` blob,
      `arrived` datetime NOT NULL,
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

查询某个客户端发布的消息::

    select * from mqtt_message where sender = "clientId";

MySQL 保留消息表(Retained Message Table)
----------------------------------------

*mqtt_retain* 表存储保留消息:

.. code-block:: sql
    
    DROP TABLE IF EXISTS `mqtt_retain`;
    CREATE TABLE `mqtt_retain` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `topic` varchar(200) DEFAULT NULL,
      `msgid` varchar(60) DEFAULT NULL,
      `sender` varchar(100) DEFAULT NULL,
      `node` varchar(100) DEFAULT NULL,
      `qos` int(2) DEFAULT NULL,
      `payload` blob,
      `arrived` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
      PRIMARY KEY (`id`),
      UNIQUE KEY `mqtt_retain_key` (`topic`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

查询retain消息::

    select * from mqtt_retain where topic = "topic0";

MySQL 接收消息ACK表(Message Acked Table)
----------------------------------------

.. code-block:: sql
    
    DROP TABLE IF EXISTS `mqtt_acked`;
    CREATE TABLE `mqtt_acked` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `clientid` varchar(200) DEFAULT NULL,
      `topic` varchar(200) DEFAULT NULL,
      `mid` int(200) DEFAULT NULL,
      `created` timestamp NULL DEFAULT NULL,
      PRIMARY KEY (`id`),
      UNIQUE KEY `mqtt_acked_key` (`clientid`,`topic`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

.. _postgre_backend:

---------------
Postgre数据存储
---------------

配置PostgreSQL存储插件
----------------------

配置文件etc/plugins/emq_backend_pgsql.conf:

.. code-block:: properties

    ## Pgsql Server
    backend.pgsql.pool1.server = 127.0.0.1:5432

    ## Pgsql Pool Size
    backend.pgsql.pool1.pool_size = 8

    ## Pgsql Username
    backend.pgsql.pool1.username = root

    ## Pgsql Password
    backend.pgsql.pool1.password = public

    ## Pgsql Database
    backend.pgsql.pool1.database = mqtt

    ## Pgsql Ssl
    backend.pgsql.pool1.ssl = false  

    ## Client Connected Record Rule
    backend.pgsql.rule.1 = {"action": "on_client_connected",   "pool": "pool1"}

    ## Subscribe Lookup Record Rule
    backend.pgsql.rule.2 = {"action": "on_subscribe_lookup",   "pool": "pool1"}

    ## Client DisConnected Record Rule
    backend.pgsql.rule.3 = {"action": "on_client_disconnected", "pool": "pool1"}

    ## Lookup Unread Message Rule
    backend.pgsql.rule.4 = {"action": "on_message_fetch",      "filter": "#", "pool": "pool1"}

    ## Lookup Retain Message Rule
    backend.pgsql.rule.5 = {"action": "on_retain_lookup",      "filter": "#", "pool": "pool1"}

    ## Store Publish Message Rule, QOS > 0
    backend.pgsql.rule.6 = {"action": "on_message_publish",    "filter": "#", "pool": "pool1"}

    ## Store Retain Message Rule
    backend.pgsql.rule.7 = {"action": "on_message_retain",     "filter": "#", "pool": "pool1"}

    ## Delete Retain Message Rule
    backend.pgsql.rule.8 = {"action": "on_retain_delete",      "filter": "#", "pool": "pool1"}

    ## Store Ack Rule
    backend.pgsql.rule.9 = {"action": "on_message_acked",      "filter": "#", "pool": "pool1"}

启用PostgreSQL存储插件
----------------------

.. code-block:: bash

    ./bin/emqctl plugins load emq_backend_pgsql

导入PostgreSQL表结构
--------------------

.. code-block:: bash

    createdb mqtt -E UTF8 -e

.. NOTE:: 数据库名称可自定义

.. code-block:: sql

   \i etc/sql/emq_backend_pgsql.sql

PostgreSQL 设备状态表(State Table)
----------------------------------

.. code-block:: sql

    CREATE TABLE mqtt_state(
      id SERIAL primary key,
      clientid character varying(100),
      state integer,
      node character varying(100),
      online_at timestamp ,
      offline_at timestamp,
      created timestamp without time zone,
      UNIQUE (clientid)
    );  

查询用户状态::

    select * from mqtt_state where clientid = "test";

PostgreSQL 用户订阅主题表(Subscription Table)
---------------------------------------------

.. code-block:: sql
    
    CREATE TABLE mqtt_subscription(
      id SERIAL primary key,
      clientid character varying(100),
      topic character varying(200),
      qos integer,
      created timestamp without time zone,
      UNIQUE (clientid, topic)
    );

设备test订阅主题topic0, topic1, topic2::

    insert into mqtt_subscription(clientid, topic, qos) values("test", "topic0", 0);
    insert into mqtt_subscription(clientid, topic, qos) values("test", "topic1", 1);
    insert into mqtt_subscription(clientid, topic, qos) values("test", "topic2", 2);

PostgreSQL MQTT消息表(Message Table)
------------------------------------

*mqtt_message* 表存储发布消息:

.. code-block:: sql
    
    CREATE TABLE mqtt_message (
      id SERIAL primary key,
      msgid character varying(60),
      sender character varying(100),
      topic character varying(200),
      qos integer,
      retain integer,
      payload text,
      arrived timestamp without time zone
    );

查询发布消息::

    select * from mqtt_message where sender = "clientId";

PostgreSQL 保留消息表(Retain Message Table)
-------------------------------------------

*mqtt_retain* 存储保留消息:

.. code-block:: sql
    
    CREATE TABLE mqtt_retain(
      id SERIAL primary key,
      topic character varying(200),
      msgid character varying(60),
      sender character varying(100),
      qos integer,
      payload text,
      arrived timestamp without time zone,
      UNIQUE (topic)
    );

查询保留消息::

    select * from mqtt_retain where topic = "topic0";

PostgreSQL 接收消息ack表(Message Acked Table)
---------------------------------------------

.. code-block:: sql
    
    CREATE TABLE mqtt_acked (
      id SERIAL primary key,
      clientid character varying(100),
      topic character varying(100),
      mid integer,
      created timestamp without time zone,
      UNIQUE (clientid, topic)
    );

.. _mongodb_backend:

---------------
MongoDB数据存储
---------------

配置MongoDB存储插件
-------------------

配置文件 etc/plugins/emq_backend_mongo.conf:

.. code-block:: properties

    ## MongoDB Server
    backend.mongo.pool1.server = 127.0.0.1:27017

    ## MongoDB Pool Size
    backend.mongo.pool1.pool_size = 8

    ## MongoDB Database
    backend.mongo.pool1.database = mqtt

    ## Client Connected Record Rule
    backend.mongo.rule.1 = {"action": "on_client_connected",   "pool": "pool1"}

    ## Subscribe Lookup Record Rule
    backend.mongo.rule.2 = {"action": "on_subscribe_lookup",   "pool": "pool1"}

    ## Client DisConnected Record Rule
    backend.mongo.rule.3 = {"action": "on_client_disconnected", "pool": "pool1"}

    ## Lookup Unread Message Rule
    backend.mongo.rule.2 = {"action": "on_message_fetch",      "filter": "#", "pool": "pool1"}

    ## Lookup Retain Message Rule
    backend.mongo.rule.3 = {"action": "on_retain_lookup",      "filter": "#", "pool": "pool1"}

    ## Store Publish Message Rule, QOS > 0
    backend.mongo.rule.4 = {"action": "on_message_publish",    "filter": "#", "pool": "pool1"}

    ## Store Retain Message Rule
    backend.mongo.rule.5 = {"action": "on_message_retain",     "filter": "#", "pool": "pool1"}

    ## Delete Retain Message Rule
    backend.mongo.rule.6 = {"action": "on_retain_delete",      "filter": "#", "pool": "pool1"}

    ## Store Ack Rule
    backend.mongo.rule.7 = {"action": "on_message_acked",      "filter": "#", "pool": "pool1"}

启用MongoDB数据存储
-------------------

.. code-block:: bash

    ./bin/emqctl plugins load emq_backend_mongo

创建MongoDB数据集合
-------------------

.. code-block:: mongodb

    use mqtt
    db.createCollection("mqtt_state")
    db.createCollection("mqtt_subscription")
    db.createCollection("mqtt_message")
    db.createCollection("mqtt_retain")
    db.createCollection("mqtt_acked")

    db.mqtt_state.ensureIndex({clientid:1, node:2})
    db.mqtt_subscription.ensureIndex({clientid:1})
    db.mqtt_message.ensureIndex({sender:1, topic:2})
    db.mqtt_retain.ensureIndex({topic:1})

.. NOTE:: 数据库名称可自定义

MongoDB 设备状态集合(State Collection)
--------------------------------------

.. code-block:: javascript

    {
        clientid: string,
        state: 0,1, //0离线 1在线
        node: string,
        online_at: timestamp,
        offline_at: timestamp
    }

查询用户状态::

    db.mqtt_state.findOne({clientid: "test"})

MongoDB 用户订阅主题集合(Subscription Collection)
-------------------------------------------------

.. code-block:: javascript

    {
        clientid: string,
        topic: string,
        qos: 0,1,2
    }

设备test分别订阅主题topic0, topic1, topic2::

    db.mqtt_subscription.insert({clientid: "test", topic: "topic0", qos: 0})
    db.mqtt_subscription.insert({clientid: "test", topic: "topic1", qos: 1})
    db.mqtt_subscription.insert({clientid: "test", topic: "topic2", qos: 2})

MongoDB 发布消息集合(Message Collection)
----------------------------------------

.. code-block:: javascript

    {
        _id: int,
        topic: string,
        msgid: string, 
        sender: string, 
        qos: 0,1,2, 
        retain: boolean (true, false),
        payload: string,
        arrived: timestamp
    }

查询发布的消息::

    db.mqtt_message.find({sender: "test"})

MongoDB 保留消息集合(Retain Message Collection)
-----------------------------------------------

.. code-block:: javascript

    {
        topic: string,
        msgid: string, 
        sender: string, 
        qos: 0,1,2, 
        payload: string,
        arrived: timestamp
    }

查询retain消息::

    db.mqtt_retain.findOne({topic: "topic0"})

MongoDB 接收消息ACK集合(Message Acked Collection)
-------------------------------------------------

.. code-block:: javascript

    {
        clientid: string, 
        topic: string, 
        mongo_id: int
    }

