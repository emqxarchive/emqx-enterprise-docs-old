
.. _authentication:

========
认证鉴权
========

------------
MQTT认证设计
------------

EMQ X认证鉴权由一系列认证插件(Plugin)提供，系统支持按用户名密码、ClientID或匿名认证，支持与MySQL、PostgreSQL、Redis、MongoDB、HTTP、LDAP集成认证。

系统默认开启匿名认证(anonymous)，通过加载认证插件可开启的多个认证模块组成认证链:

.. image:: _static/images/7.png

------------
匿名认证设置
------------

匿名认证是认证链条最后一个环节， etc/emqx.conf中默认启用匿名认证，产品部署建议关闭:

.. code-block:: properties

    ## Allow Anonymous authentication
    mqtt.allow_anonymous = true

-------------
访问控制(ACL)
-------------

EMQ X消息服务器通过ACL(Access Control List)实现MQTT客户端访问控制。

ACL访问控制规则定义::

    允许(Allow)|拒绝(Deny) 谁(Who) 订阅(Subscribe)|发布(Publish) 主题列表(Topics)

MQTT客户端发起订阅/发布请求时，EMQ X消息服务器的访问控制模块，会逐条匹配ACL规则，直到匹配成功为止:

.. image:: _static/images/6.png

----------------
默认访问控制设置
----------------

EMQ X消息服务器默认访问控制，通过acl.conf配置文件设置:

.. code-block:: properties
    
    ## ACL nomatch
    mqtt.acl_nomatch = allow

    ## Default ACL File
    mqtt.acl_file = etc/acl.conf

ACL规则定义在etc/acl.conf，EMQ X启动时加载到内存:

.. code-block:: erlang

    %% Allow 'dashboard' to subscribe '$SYS/#'
    {allow, {user, "dashboard"}, subscribe, ["$SYS/#"]}.

    %% Allow clients from localhost to subscribe any topics
    {allow, {ipaddr, "127.0.0.1"}, pubsub, ["$SYS/#", "#"]}.

    %% Deny clients to subscribe '$SYS#' and '#'
    {deny, all, subscribe, ["$SYS/#", {eq, "#"}]}.


ACL规则修改后可通过命令行重新加载:

.. code-block:: console

    $ ./bin/emqx_ctl acl reload

    reload acl_internal successfully

------------
认证插件列表
------------

EMQ X支持ClientId、用户名、HTTP、LDAP、MySQL、Redis、Postgre、MongoDB多种认集成方式，以认证插件方式提供可同时加载多个形成认证链。

EMQ X认证插件配置文件，在/etc/emqx/plugins/(RPM/DEB安装)或etc/plugins/(独立安装)目录:

+-------------------------+---------------------------+---------------------------+
| 认证插件                | 配置文件                  | 说明                      |
+=========================+===========================+===========================+
| emqx_auth_clientid      | emqx_auth_clientid.conf   | ClientId认证/鉴权插件     |
+-------------------------+---------------------------+---------------------------+
| emqx_auth_username      | emqx_auth_username.conf   | 用户名密码认证/鉴权插件   |
+-------------------------+---------------------------+---------------------------+
| emqx_auth_ldap          | emqx_auth_ldap.conf       | LDAP认证/鉴权插件         |
+-------------------------+---------------------------+---------------------------+
| emqx_auth_http          | emqx_auth_http.conf       | HTTP认证/鉴权插件         |
+-------------------------+---------------------------+---------------------------+
| emqx_auth_mysql         | emqx_auth_redis.conf      | MySQL认证/鉴权插件        |
+-------------------------+---------------------------+---------------------------+
| emqx_auth_pgsql         | emqx_auth_mysql.conf      | Postgre认证/鉴权插件      |
+-------------------------+---------------------------+---------------------------+
| emqx_auth_redis         | emqx_auth_pgsql.conf      | Redis认证/鉴权插件        |
+-------------------------+---------------------------+---------------------------+
| emqx_auth_mongo         | emqx_auth_mongo.conf      | MongoDB认证/鉴权插件      |
+-------------------------+---------------------------+---------------------------+

--------------------
ClientID认证插件配置
--------------------

配置文件emqx_auth_clientid.conf，配置ClientID、密码列表:

.. code-block:: properties

    ## auth.client.${id}.clientid = ${clientid}
    ## auth.client.${id}.password = ${password}

    ## Examples
    auth.client.1.clientid = id
    auth.client.1.password = passwd
    auth.client.2.clientid = dev:devid
    auth.client.2.password = passwd2
    auth.client.3.clientid = app:appid
    auth.client.3.password = passwd3
    auth.client.4.clientid = client~!@#$%^&*()_+
    auth.client.4.password = passwd~!@#$%^&*()_+

加载ClientId认证插件:

.. code-block:: console

    ./bin/emqx_ctl plugins load emqx_auth_clientid

------------------
用户名认证插件配置
------------------

配置文件emqx_auth_username.conf，配置用户名、密码列表:

.. code-block:: properties

    ##auth.user.$N.username = admin
    ##auth.user.$N.password = public

    ## Examples:
    ##auth.user.1.username = admin
    ##auth.user.1.password = public
    ##auth.user.2.username = feng@emqtt.io
    ##auth.user.2.password = public
    ##auth.user.3.username = name~!@#$%^&*()_+
    ##auth.user.3.password = pwsswd~!@#$%^&*()_+

加载用户名认证插件:

.. code-block:: console

    ./bin/emqx_ctl plugins load emqx_auth_username

该插件加载后，两种方式添加用户:

1. 直接在emqx_auth_username.conf中明文配置用户::

    auth.user.1.username = admin
    auth.user.1.password = public

2. 通过'./bin/emqx_ctl'管理命令行添加用户:

.. code-block:: console

   $ ./bin/emqx_ctl users add <Username> <Password>

----------------
LDAP认证插件配置
----------------

配置文件emqx_auth_ldap.conf，配置LDAP服务器参数:

.. code-block:: properties

    auth.ldap.servers = 127.0.0.1

    auth.ldap.port = 389

    auth.ldap.timeout = 30

    auth.ldap.user_dn = uid=%u,ou=People,dc=example,dc=com

    auth.ldap.ssl = false

加载LDAP认证插件:

.. code-block:: console

    ./bin/emqx_ctl plugins load emqx_auth_ldap

----------------
HTTP认证插件配置
----------------

配置文件emqx_auth_http.conf，设置认证URL与参数:

.. code-block:: properties

    ## Variables: %u = username, %c = clientid, %a = ipaddress, %P = password, %t = topic

    auth.http.auth_req = http://127.0.0.1:8080/mqtt/auth
    auth.http.auth_req.method = post
    auth.http.auth_req.params = clientid=%c,username=%u,password=%P

设置超级用户URL与参数:

.. code-block:: properties

    auth.http.super_req = http://127.0.0.1:8080/mqtt/superuser
    auth.http.super_req.method = post
    auth.http.super_req.params = clientid=%c,username=%u

设置访问控制(ACL)URL与参数:

.. code-block:: properties

    ## 'access' parameter: sub = 1, pub = 2
    auth.http.acl_req = http://127.0.0.1:8080/mqtt/acl
    auth.http.acl_req.method = get
    auth.http.acl_req.params = access=%A,username=%u,clientid=%c,ipaddr=%a,topic=%t

HTTP认证/访问控制(ACL)服务器API设计::

    认证/ACL成功，API返回200

    认证/ACL失败，API返回4xx

加载HTTP认证插件:

.. code-block:: console

    ./bin/emqx_ctl plugins load emqx_auth_http

-----------------
MySQL认证插件配置
-----------------

配置文件emqx_auth_mysql.conf, 默认的MQTT用户、ACL库表和认证设置:

MQTT认证用户表
--------------

.. code-block:: sql

    CREATE TABLE `mqtt_user` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `username` varchar(100) DEFAULT NULL,
      `password` varchar(100) DEFAULT NULL,
      `salt` varchar(35) DEFAULT NULL,
      `is_superuser` tinyint(1) DEFAULT 0,
      `created` datetime DEFAULT NULL,
      PRIMARY KEY (`id`),
      UNIQUE KEY `mqtt_username` (`username`)
    ) ENGINE=MyISAM DEFAULT CHARSET=utf8;

.. NOTE:: 用户可自定义认证用户表，通过'authquery'配置查询语句。

MQTT访问控制表
--------------

.. code-block:: sql

    CREATE TABLE `mqtt_acl` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `allow` int(1) DEFAULT NULL COMMENT '0: deny, 1: allow',
      `ipaddr` varchar(60) DEFAULT NULL COMMENT 'IpAddress',
      `username` varchar(100) DEFAULT NULL COMMENT 'Username',
      `clientid` varchar(100) DEFAULT NULL COMMENT 'ClientId',
      `access` int(2) NOT NULL COMMENT '1: subscribe, 2: publish, 3: pubsub',
      `topic` varchar(100) NOT NULL DEFAULT '' COMMENT 'Topic Filter',
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

    INSERT INTO `mqtt_acl` (`id`, `allow`, `ipaddr`, `username`, `clientid`, `access`, `topic`)
    VALUES
        (1,1,NULL,'$all',NULL,2,'#'),
        (2,0,NULL,'$all',NULL,1,'$SYS/#'),
        (3,0,NULL,'$all',NULL,1,'eq #'),
        (5,1,'127.0.0.1',NULL,NULL,2,'$SYS/#'),
        (6,1,'127.0.0.1',NULL,NULL,2,'#'),
        (7,1,NULL,'dashboard',NULL,1,'$SYS/#');

配置MySQL服务器地址
-------------------

.. code-block:: properties

    ## Mysql Server 3306, 127.0.0.1:3306, localhost:3306
    auth.mysql.server = 127.0.0.1:3306

    ## Mysql Pool Size
    auth.mysql.pool = 8

    ## Mysql Username
    ## auth.mysql.username = 

    ## Mysql Password
    ## auth.mysql.password = 

    ## Mysql Database
    auth.mysql.database = mqtt

配置MySQL认证查询语句
---------------------

.. code-block:: properties

    ## Variables: %u = username, %c = clientid

    ## Authentication Query: select password or password,salt
    auth.mysql.auth_query = select password from mqtt_user where username = '%u' limit 1

    ## Password hash: plain, md5, sha, sha256, pbkdf2, bcrypt
    auth.mysql.password_hash = sha256

    ## sha256 with salt prefix
    ## auth.mysql.password_hash = salt sha256

    ## sha256 with salt suffix
    ## auth.mysql.password_hash = sha256 salt

    ## %% Superuser Query
    auth.mysql.super_query = select is_superuser from mqtt_user where username = '%u' limit 1

配置MySQL访问控制查询语句
-------------------------

.. code-block:: properties

    ## ACL Query Command
    auth.mysql.acl_query = select allow, ipaddr, username, clientid, access, topic from mqtt_acl where ipaddr = '%a' or username = '%u' or username = '$all' or clientid = '%c'

加载MySQL认证插件
-----------------

.. code-block:: console

    ./bin/emqx_ctl plugins load emqx_auth_mysql

---------------------
Postgre认证制插件配置
---------------------

配置文件emqx_auth_pgsql.conf, 默认的MQTT用户、ACL库表和认证设置:

Postgre MQTT用户表
------------------

.. code-block:: sql

    CREATE TABLE mqtt_user (
      id SERIAL primary key,
      is_superuser boolean,
      username character varying(100),
      password character varying(100),
      salt character varying(40)
    );

.. NOTE:: 用户可自定义认证用户表，通过'authquery'配置查询语句。

Postgre MQTT访问控制表
----------------------

.. code-block:: sql

    CREATE TABLE mqtt_acl (
      id SERIAL primary key,
      allow integer,
      ipaddr character varying(60),
      username character varying(100),
      clientid character varying(100),
      access  integer,
      topic character varying(100)
    );

    INSERT INTO mqtt_acl (id, allow, ipaddr, username, clientid, access, topic)
    VALUES
        (1,1,NULL,'$all',NULL,2,'#'),
        (2,0,NULL,'$all',NULL,1,'$SYS/#'),
        (3,0,NULL,'$all',NULL,1,'eq #'),
        (5,1,'127.0.0.1',NULL,NULL,2,'$SYS/#'),
        (6,1,'127.0.0.1',NULL,NULL,2,'#'),
        (7,1,NULL,'dashboard',NULL,1,'$SYS/#');

配置Postgre服务器地址
---------------------

.. code-block:: properties

    ## Postgre Server
    auth.pgsql.server = 127.0.0.1:5432

    auth.pgsql.pool = 8

    auth.pgsql.username = root

    #auth.pgsql.password = 

    auth.pgsql.database = mqtt

    auth.pgsql.encoding = utf8

    auth.pgsql.ssl = false

配置Postgre认证查询语句
-----------------------

.. code-block:: properties

    ## Variables: %u = username, %c = clientid, %a = ipaddress

    ## Authentication Query: select password or password,salt
    auth.pgsql.auth_query = select password from mqtt_user where username = '%u' limit 1

    ## Password hash: plain, md5, sha, sha256, pbkdf2, bcrypt
    auth.pgsql.password_hash = sha256

    ## sha256 with salt prefix
    ## auth.pgsql.password_hash = salt sha256

    ## sha256 with salt suffix
    ## auth.pgsql.password_hash = sha256 salt

    ## Superuser Query
    auth.pgsql.super_query = select is_superuser from mqtt_user where username = '%u' limit 1

配置Postgre访问控制语句
-----------------------

.. code-block:: properties

    ## ACL Query. Comment this query, the acl will be disabled.
    auth.pgsql.acl_query = select allow, ipaddr, username, clientid, access, topic from mqtt_acl where ipaddr = '%a' or username = '%u' or username = '$all' or clientid = '%c'

加载Postgre认证插件
-------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_auth_pgsql

-----------------
Redis认证插件配置
-----------------

配置文件emqx_auth_redis.conf:

配置Redis服务器地址
-------------------

.. code-block:: properties

    ## Redis Server
    auth.redis.server = 127.0.0.1:6379

    ## Redis Pool Size
    auth.redis.pool = 8

    ## Redis Database
    auth.redis.database = 0

    ## Redis Password
    ## auth.redis.password =

配置认证查询命令
----------------

.. code-block:: properties

    ## Variables: %u = username, %c = clientid

    ## Authentication Query Command
    ## HMGET mqtt_user:%u password or HMGET mqtt_user:%u password salt or HGET mqtt_user:%u password
    auth.redis.auth_cmd = HGET mqtt_user:%u password

    ## Password hash: plain, md5, sha, sha256, pbkdf2, bcrypt
    auth.redis.passwd.hash = sha256

    ## Superuser Query Command
    auth.redis.super_cmd = HGET mqtt_user:%u is_superuser

配置访问控制查询命令
--------------------

.. code-block:: properties

    ## ACL Query Command
    auth.redis.acl_cmd = HGETALL mqtt_acl:%u

Redis认证用户Hash
-----------------

默认采用Hash存储认证用户::

    HSET mqtt_user:<username> is_superuser 1
    HSET mqtt_user:<username> password "passwd"

Redis ACL规则Hash
-----------------

默认采用Hash存储ACL规则::

    HSET mqtt_acl:<username> topic1 1
    HSET mqtt_acl:<username> topic2 2
    HSET mqtt_acl:<username> topic3 3

.. NOTE:: 1: subscribe, 2: publish, 3: pubsub

加载Redis认证插件
-----------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_auth_redis

-------------------
MongoDB认证插件配置
-------------------

配置文件emqx_auth_mongo.conf, MongoDB、MQTT用户、ACL集合设置:

配置MongoDB服务器
-----------------

.. code-block:: properties

    ## Mongo Server
    auth.mongo.server = 127.0.0.1:27017

    ## Mongo Pool Size
    auth.mongo.pool = 8

    ## Mongo User
    ## auth.mongo.user = 

    ## Mongo Password
    ## auth.mongo.password = 

    ## Mongo Database
    auth.mongo.database = mqtt

配置认证查询集合
----------------

.. code-block:: properties

    ## authquery
    auth.mongo.authquery.collection = mqtt_user

    auth.mongo.authquery.password_field = password

    auth.mongo.authquery.password_hash = sha256

    auth.mongo.authquery.selector = username=%u

    ## superquery
    auth.mongo.superquery.collection = mqtt_user

    auth.mongo.superquery.super_field = is_superuser

    auth.mongo.superquery.selector = username=%u

    ## acl_query
    auth.mongo.acl_query.collection = mqtt_user

    auth.mongo.acl_query.selector = username=%u

配置ACL查询集合
---------------

.. code-block:: properties

    ## aclquery
    auth.mongo.aclquery.collection = mqtt_acl

    auth.mongo.aclquery.selector = username=%u


MongoDB数据库
-------------

.. code-block:: console

    use mqtt
    db.createCollection("mqtt_user")
    db.createCollection("mqtt_acl")
    db.mqtt_user.ensureIndex({"username":1})

.. NOTE:: 数据库、集合名称可自定义

MongoDB 用户集合示例
--------------------

.. code-block:: javascript

    {
        username: "user",
        password: "password hash",
        is_superuser: boolean (true, false),
        created: "datetime"
    }

    db.mqtt_user.insert({username: "test", password: "password hash", is_superuser: false})
    db.mqtt_user:insert({username: "root", is_superuser: true})

MongoDB ACL集合示例
-------------------

.. code-block:: javascript

    {
        username: "username",
        clientid: "clientid",
        publish: ["topic1", "topic2", ...],
        subscribe: ["subtop1", "subtop2", ...],
        pubsub: ["topic/#", "topic1", ...]
    }

    db.mqtt_acl.insert({username: "test", publish: ["t/1", "t/2"], subscribe: ["user/%u", "client/%c"]})
    db.mqtt_acl.insert({username: "admin", pubsub: ["#"]})

加载Mognodb认证插件
-------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_auth_mongo

.. _recon: http://ferd.github.io/recon/


