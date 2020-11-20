# AuthN/AuthZ

## Design of MQTT Auth

EMQ X utilizes plugins to provide various Authentication mechanism. EMQ X supports username / password, ClientID and anonymous authentication. It also supports Auth integration with MySQL, PostgreSQL, Redis, MongoDB, HTTP, OpenLDAP and JWT.

Anonymous Auth is enabled by default. An Auth chain can be built of multiple Auth plugins:

![image](./_static/images/authn_1.png)

## Anonymous Auth

Anonymous Auth is the last validation of Auth chain. By default, it is enabled in file etc/emqx.conf . Recommended to disable it in production deployment:

    ## Allow Anonymous authentication
    mqtt.allow_anonymous = true

## Access Control List (ACL)

EMQ X Server utilizes Access Control List (ACL) to realize the access control upon clients.

ACL defines:

    Allow|Deny Whom Subscribe|Publish Topics

When MQTT clients subscribe to topics or publish messages, the EMQ X access control module tries to check against all rules in the list till a first match. Otherwise it fallbacks to default routine if no match found:

![image](./_static/images/authn_2.png)

## Default Access Control File

The default access control of EMQ X server is configured by the file acl.conf:

    ## ACL nomatch
    acl_nomatch = allow

    ## Default ACL File
    acl_file = etc/acl.conf

    ## Whether to enable ACL cache.
    enable_acl_cache = on

    ## The maximum count of ACL entries can be cached for a client.
    acl_cache_max_size = 32

    ## The time after which an ACL cache entry will be deleted
    acl_cache_ttl = 1m

    ## The action when acl check reject current operation
    acl_deny_action = ignore

ACL is defined in etc/acl.conf , and is loaded when EMQ X starts:

    %% Allow 'dashboard' to subscribe '$SYS/#'
    {allow, {user, "dashboard"}, subscribe, ["$SYS/#"]}.

    %% Allow clients from localhost to subscribe any topics
    {allow, {ipaddr, "127.0.0.1"}, pubsub, ["$SYS/#", "#"]}.

    %% Deny clients to subscribe '$SYS#' and '#'
    {deny, all, subscribe, ["$SYS/#", {eq, "#"}]}.

    %% Allow all
    {allow, all}.

If ACL is modified, it can be reloaded using CLI:

    $ ./bin/emqx_ctl acl reload

    reload acl_internal successfully

## List of AuthN/ACL Plugins

EMQ X supports integrated authentications by using ClientId, Username, HTTP, OpenLDAP, MySQL, Redis, PostgreSQL, MongoDB and JWT. Multiple Auth plugins can be loaded simultaneously to build an authentication chain.

The config files of Auth plugins are located in /etc/emqx/plugins/ (RPM/DEB installation) or in etc/plugins/ (standalone installation)

| Auth Plugin        | Config file             | Description                    |
| ------------------ | ----------------------- | ------------------------------ |
| emqx_auth_clientid | emqx_auth_clientid.conf | ClientId AuthN Plugin          |
| emqx_auth_username | emqx_auth_username.conf | Username/Password AuthN Plugin |
| emqx_auth_ldap     | emqx_auth_ldap.conf     | OpenLDAP AuthN/AuthZ Plugin    |
| emqx_auth_http     | emqx_auth_http.conf     | HTTP AuthN/AuthZ               |
| emqx_auth_mysql    | emqx_auth_redis.conf    | MySQL AuthN/AuthZ              |
| emqx_auth_pgsql    | emqx_auth_mysql.conf    | PostgreSQL AuthN/AuthZ         |
| emqx_auth_redis    | emqx_auth_pgsql.conf    | Redis AuthN/AuthZ              |
| emqx_auth_mongo    | emqx_auth_mongo.conf    | MongoDB AuthN/AuthZ            |
| emqx_auth_jwt      | emqx_auth_jwt.conf      | JWT AuthN/AuthZ                |

## ClientID Auth Plugin

Configure the password hash in the `emqx_auth_clientid.conf`:

    ## Default usernames Examples
    ##auth.client.1.clientid = id
    ##auth.client.1.password = passwd
    ##auth.client.2.clientid = dev:devid
    ##auth.client.2.password = passwd2
    ##auth.client.3.clientid = app:appid
    ##auth.client.3.password = passwd3
    ##auth.client.4.clientid = client~!@#$%^&*()_+
    ##auth.client.4.password = passwd~!@#$%^&*()_+
    ## Password hash: plain | md5 | sha | sha256
    auth.client.password_hash = sha256

Load ClientId Auth plugin:

    ./bin/emqx_ctl plugins load emqx_auth_clientid

After the plugin is loaded, there are two possible ways to add users:

1.  Use the './bin/emqx_ctl' CLI tool to add clients:

        $ ./bin/emqx_ctl clientid add \<ClientId> \<Password>

2.  Use the HTTP API to add clients:

        POST api/v3/auth_clientid

        {
         "clientid": "clientid",
         "password": "password"
        }

## Username/Passwordd Auth Plugin

Configure the password hash in the `emqx_auth_username.conf`:

    ## Default usernames Examples:
    ##auth.user.1.username = admin
    ##auth.user.1.password = public
    ##auth.user.2.username = feng@emqtt.io
    ##auth.user.2.password = public
    ##auth.user.3.username = name~!@#$%^&*()_+
    ##auth.user.3.password = pwsswd~!@#$%^&*()_+
    ## Password hash: plain | md5 | sha | sha256
    auth.user.password_hash = sha256

Load Username Auth plugin:

    ./bin/emqx_ctl plugins load emqx_auth_username

After the plugin is loaded, there are two possible ways to add users:

1.  Use the './bin/emqx_ctl' CLI tool to add users:

        $ ./bin/emqx_ctl users add \<Username> \<Password>

2.  Use the HTTP API to add users:

        POST api/v3/auth_username

        {
          "username": "username",
          "password": "password"
        }

## OpenLDAP Auth Plugin

Configure the OpenLDAP Auth Plugin in the `emqx_auth_ldap.conf`:

    ## OpenLDAP servers list
    auth.ldap.servers = 127.0.0.1

    ## OpenLDAP server port
    auth.ldap.port = 389

    ## OpenLDAP pool size
    auth.ldap.pool = 8

    ## OpenLDAP Bind DN
    auth.ldap.bind_dn = cn=root,dc=emqx,dc=io

    ## OpenLDAP Bind Password
    auth.ldap.bind_password = public

    ## OpenLDAP query timeout
    auth.ldap.timeout = 30s

    ## OpenLDAP Device DN
    auth.ldap.device_dn = ou=device,dc=emqx,dc=io

    ## Specified ObjectClass
    auth.ldap.match_objectclass = mqttUser

    ## attributetype for username
    auth.ldap.username.attributetype = uid

    ## attributetype for password
    auth.ldap.password.attributetype = userPassword

    ## Whether to enable SSL
    auth.ldap.ssl = false

Load the OpenLDAP Auth plugin:

    ./bin/emqx_ctl plugins load emqx_auth_ldap

## HTTP Auth/ACL Plugin

Configure the HTTP Auth/ACL Plugin in the `emqx_auth_http.conf`:

    ## Time-out time for the http request, 0 is never timeout
    ## auth.http.request.timeout = 0

    ## Connection time-out time, used during the initial request
    ## when the client is connecting to the server
    ## auth.http.request.connect_timout = 0

    ## Re-send http reuqest times
    auth.http.request.retry_times = 3

    ## The interval for re-sending the http request
    auth.http.request.retry_interval = 1s

    ## The 'Exponential Backoff' mechanism for re-sending request. The actually
    ## re-send time interval is `interval * backoff ^ times`
    auth.http.request.retry_backoff = 2.0

Setup the Auth URL and parameters:

    ## Variables: %u = username, %c = clientid, %a = ipaddress, %P = password, %t = topic

    auth.http.auth_req = http://127.0.0.1:8080/mqtt/auth
    auth.http.auth_req.method = post
    auth.http.auth_req.params = clientid=%c,username=%u,password=%P

Setup the Super User URL and parameters:

    auth.http.super_req = http://127.0.0.1:8080/mqtt/superuser
    auth.http.super_req.method = post
    auth.http.super_req.params = clientid=%c,username=%u

Setup the ACL URL and parameters:

    ## 'access' parameter: sub = 1, pub = 2
    auth.http.acl_req = http://127.0.0.1:8080/mqtt/acl
    auth.http.acl_req.method = get
    auth.http.acl_req.params = access=%A,username=%u,clientid=%c,ipaddr=%a,topic=%t

Design of HTTP Auth and ACL server API:

    If Auth/ACL sucesses, API returns 200

    If Auth/ACL fails, API return 4xx

Load HTTP Auth/ACL plugin:

    ./bin/emqx_ctl plugins load emqx_auth_http

## MySQL Auth/ACL Plugin

Create MQTT users' ACL database, and configure the ACL and Auth queries in the `emqx_auth_mysql.conf`:

### MQTT Auth User List

    CREATE TABLE `mqtt_user` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `username` varchar(100) DEFAULT NULL,
      `password` varchar(100) DEFAULT NULL,
      `salt` varchar(40) DEFAULT NULL,
      `is_superuser` tinyint(1) DEFAULT 0,
      `created` datetime DEFAULT NULL,
      PRIMARY KEY (`id`),
      UNIQUE KEY `mqtt_username` (`username`)
    ) ENGINE=MyISAM DEFAULT CHARSET=utf8;

::: tip Tip
User can define the user list table and configure it in the `auth_query` statement.
:::

### MQTT Access Control List

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
        (4,1,'127.0.0.1',NULL,NULL,2,'$SYS/#'),
        (5,1,'127.0.0.1',NULL,NULL,2,'#'),
        (6,1,NULL,'dashboard',NULL,1,'$SYS/#');

### MySQL Server Address

    ## MySQL Server
    auth.mysql.server = 127.0.0.1:3306

    ## MySQL Pool Size
    auth.mysql.pool = 8

    ## MySQL Username
    ## auth.mysql.username =

    ## MySQL Password
    ## auth.mysql.password =

    ## MySQL Database
    auth.mysql.database = mqtt

    ## MySQL query timeout
    ## auth.mysql.query_timeout = 5s

### Configure MySQL Auth Query Statement

    ## Authentication query
    ##
    ## Variables:
    ##  - %u: username
    ##  - %c: clientid
    ##  - %C: common name of client TLS cert
    ##  - %d: subject of client TLS cert
    ##
    auth.mysql.auth_query = select password from mqtt_user where username = '%u' limit 1
    ## auth.mysql.auth_query = select password_hash as password from mqtt_user where username = '%u' limit 1

    ## Password hash: plain | md5 | sha | sha256 | bcrypt
    auth.mysql.password_hash = sha256

    ## sha256 with salt prefix
    ## auth.mysql.password_hash = salt,sha256

    ## bcrypt with salt only prefix
    ## auth.mysql.password_hash = salt,bcrypt

    ## sha256 with salt suffix
    ## auth.mysql.password_hash = sha256,salt

    ## pbkdf2 with macfun iterations dklen
    ## macfun: md4, md5, ripemd160, sha, sha224, sha256, sha384, sha512
    ## auth.mysql.password_hash = pbkdf2,sha256,1000,20

    ## Superuser query
    ##
    ## Variables:
    ##  - %u: username
    ##  - %c: clientid
    ##  - %C: common name of client TLS cert
    ##  - %d: subject of client TLS cert
    auth.mysql.super_query = select is_superuser from mqtt_user where username = '%u' limit 1

### Configure MySQL ACL Query Statement

    ## ACL query
    ##
    ## Variables:
    ##  - %a: ipaddr
    ##  - %u: username
    ##  - %c: clientid
    auth.mysql.acl_query = select allow, ipaddr, username, clientid, access, topic from mqtt_acl where ipaddr = '%a' or username = '%u' or username = '$all' or clientid = '%c'

### Load MySQL Auth Plugin

    ./bin/emqx_ctl plugins load emqx_auth_mysql

## PostgreSQL Auth/ACL Plugin

Create MQTT users' ACL tables, and configure Auth, ACL queries in the `emqx_auth_pgsql.conf`:

### PostgreSQL MQTT User Table

    CREATE TABLE mqtt_user (
      id SERIAL primary key,
      is_superuser boolean,
      username character varying(100),
      password character varying(100),
      salt character varying(40)
    );

::: tip Tip
User can define the user list table and configure it in the auth_query statement.
:::

### PostgreSQL MQTT ACL Table

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
        (4,1,'127.0.0.1',NULL,NULL,2,'$SYS/#'),
        (5,1,'127.0.0.1',NULL,NULL,2,'#'),
        (6,1,NULL,'dashboard',NULL,1,'$SYS/#');

### PostgreSQL Server Address

    ## PostgreSQL Server
    auth.pgsql.server = 127.0.0.1:5432

    ## PostgreSQL pool size
    auth.pgsql.pool = 8

    ## PostgreSQL username
    auth.pgsql.username = root

    ## PostgreSQL password
    #auth.pgsql.password =

    ## PostgreSQL database
    auth.pgsql.database = mqtt

    ## PostgreSQL database encoding
    auth.pgsql.encoding = utf8

    ## Whether to enable SSL connection
    auth.pgsql.ssl = false

    ## SSL keyfile
    ## auth.pgsql.ssl_opts.keyfile =

    ## SSL certfile
    ## auth.pgsql.ssl_opts.certfile =

    ## SSL cacertfile
    ## auth.pgsql.ssl_opts.cacertfile =

### Configure PostgreSQL Auth Query Statement

    ## Authentication query
    ##
    ## Variables:
    ##  - %u: username
    ##  - %c: clientid
    ##  - %C: common name of client TLS cert
    ##  - %d: subject of client TLS cert
    auth.pgsql.auth_query = select password from mqtt_user where username = '%u' limit 1

    ## Password hash: plain | md5 | sha | sha256 | bcrypt
    auth.pgsql.password_hash = sha256

    ## sha256 with salt prefix
    ## auth.pgsql.password_hash = salt,sha256

    ## sha256 with salt suffix
    ## auth.pgsql.password_hash = sha256,salt

    ## bcrypt with salt prefix
    ## auth.pgsql.password_hash = salt,bcrypt

    ## pbkdf2 with macfun iterations dklen
    ## macfun: md4, md5, ripemd160, sha, sha224, sha256, sha384, sha512
    ## auth.pgsql.password_hash = pbkdf2,sha256,1000,20

    ## Superuser query
    ##
    ## Variables:
    ##  - %u: username
    ##  - %c: clientid
    ##  - %C: common name of client TLS cert
    ##  - %d: subject of client TLS cert
    auth.pgsql.super_query = select is_superuser from mqtt_user where username = '%u' limit 1

### Configure PostgreSQL ACL Query Statement

    ## ACL query. Comment this query, the ACL will be disabled
    ##
    ## Variables:
    ##  - %a: ipaddress
    ##  - %u: username
    ##  - %c: clientid
    auth.pgsql.acl_query = select allow, ipaddr, username, clientid, access, topic from mqtt_acl where ipaddr = '%a' or username = '%u' or username = '$all' or clientid = '%c'

### Load PostgreSQL Auth Plugin

    ./bin/emqx_ctl plugins load emqx_auth_pgsql

## Redis/ACL Auth Plugin

Config file: `emqx_auth_redis.conf`:

### Redis Server Address

    ## Redis Server cluster type: single | sentinel | cluster
    auth.redis.type = single

    ## Redis server address
    auth.redis.server = 127.0.0.1:6379

    ## Redis sentinel cluster name.
    ## auth.redis.sentinel = mymaster

    ## Redis pool size.
    auth.redis.pool = 8

    ## Redis database no.
    auth.redis.database = 0

    ## Redis password
    ## auth.redis.password =

    ## Redis query timeout
    ## auth.redis.query_timeout = 5s

### Configure Auth Query Command

    ## Authentication query command
    ##
    ## Variables:
    ##  - %u: username
    ##  - %c: clientid
    ##  - %C: common name of client TLS cert
    ##  - %d: subject of client TLS cert
    auth.redis.auth_cmd = HMGET mqtt_user:%u password

    ## Password hash: plain | md5 | sha | sha256 | bcrypt
    auth.redis.password_hash = plain

    ## sha256 with salt prefix
    ## auth.redis.password_hash = salt,sha256

    ## sha256 with salt suffix
    ## auth.redis.password_hash = sha256,salt

    ## bcrypt with salt prefix
    ## auth.redis.password_hash = salt,bcrypt

    ## pbkdf2 with macfun iterations dklen
    ## macfun: md4, md5, ripemd160, sha, sha224, sha256, sha384, sha512
    ## auth.redis.password_hash = pbkdf2,sha256,1000,20

    ## Superuser query command
    ##
    ## Variables:
    ##  - %u: username
    ##  - %c: clientid
    ##  - %C: common name of client TLS cert
    ##  - %d: subject of client TLS cert
    auth.redis.super_cmd = HGET mqtt_user:%u is_superuser

### Configure ACL Query Command

    ## ACL Query Command
    ##
    ## Variables:
    ##  - %u: username
    ##  - %c: clientid
    auth.redis.acl_cmd = HGETALL mqtt_acl:%u

### Redis Authed Users Hash

By default, Hash is used to store Authed users:

    HSET mqtt_user:\<username> is_superuser 1
    HSET mqtt_user:\<username> password "passwd"

### Redis ACL Rules Hash

By default, Hash is used to store ACL rules:

    HSET mqtt_acl:\<username> topic1 1
    HSET mqtt_acl:\<username> topic2 2
    HSET mqtt_acl:\<username> topic3 3

::: tip Tip
1: subscribe, 2: publish, 3: pubsub
:::

### Load Redis Auth Plugin

    ./bin/emqx_ctl plugins load emqx_auth_redis

## MongoDB Auth/ACL Plugin

Configure MongoDB, MQTT users and ACL Collection in the `emqx_auth_mongo.conf`:

### MongoDB Server

    ## MongoDB Topology Type: single | unknown | sharded | rs
    auth.mongo.type = single

    ## The set name if type is rs
    ## auth.mongo.rs_set_name =

    ## MongoDB server list.
    auth.mongo.server = 127.0.0.1:27017

    ## MongoDB pool size
    auth.mongo.pool = 8

    ## MongoDB login user
    ## auth.mongo.login =

    ## MongoDB password
    ## auth.mongo.password =

    ## MongoDB AuthSource
    ## auth.mongo.auth_source = admin

    ## MongoDB database
    auth.mongo.database = mqtt

    ## MongoDB query timeout
    ## auth.mongo.query_timeout = 5s

    ## Whether to enable SSL connection
    ## auth.mongo.ssl = false

    ## SSL keyfile
    ## auth.mongo.ssl_opts.keyfile =

    ## SSL certfile
    ## auth.mongo.ssl_opts.certfile =

    ## SSL cacertfile
    ## auth.mongo.ssl_opts.cacertfile =

    ## MongoDB write mode
    ## auth.mongo.w_mode =

    ## Mongo read mode
    ## auth.mongo.r_mode =

    ## MongoDB topology options
    auth.mongo.topology.pool_size = 1
    auth.mongo.topology.max_overflow = 0
    ## auth.mongo.topology.overflow_ttl = 1000
    ## auth.mongo.topology.overflow_check_period = 1000
    ## auth.mongo.topology.local_threshold_ms = 1000
    ## auth.mongo.topology.connect_timeout_ms = 20000
    ## auth.mongo.topology.socket_timeout_ms = 100
    ## auth.mongo.topology.server_selection_timeout_ms = 30000
    ## auth.mongo.topology.wait_queue_timeout_ms = 1000
    ## auth.mongo.topology.heartbeat_frequency_ms = 10000
    ## auth.mongo.topology.min_heartbeat_frequency_ms = 1000

### Configure Auth Query Collection

    ## Authentication query
    auth.mongo.auth_query.collection = mqtt_user

    ## password with salt prefix
    ## auth.mongo.auth_query.password_hash = salt,sha256
    auth.mongo.auth_query.password_field = password

    ## Password hash: plain | md5 | sha | sha256 | bcrypt
    auth.mongo.auth_query.password_hash = sha256

    ## sha256 with salt suffix
    ## auth.mongo.auth_query.password_hash = sha256,salt

    ## sha256 with salt prefix
    ## auth.mongo.auth_query.password_hash = salt,sha256

    ## bcrypt with salt prefix
    ## auth.mongo.auth_query.password_hash = salt,bcrypt

    ## pbkdf2 with macfun iterations dklen
    ## macfun: md4, md5, ripemd160, sha, sha224, sha256, sha384, sha512
    ## auth.mongo.auth_query.password_hash = pbkdf2,sha256,1000,20

    ## Authentication Selector
    ##
    ## Variables:
    ##  - %u: username
    ##  - %c: clientid
    ##  - %C: common name of client TLS cert
    ##  - %d: subject of client TLS cert
    auth.mongo.auth_query.selector = username=%u

    ## Enable superuser query
    auth.mongo.super_query = on

    auth.mongo.super_query.collection = mqtt_user

    auth.mongo.super_query.super_field = is_superuser

    #auth.mongo.super_query.selector = username=%u, clientid=%c
    auth.mongo.super_query.selector = username=%u

### Configure ACL Query Collection

    ## Enable ACL query
    auth.mongo.acl_query = on

    auth.mongo.acl_query.collection = mqtt_acl

    ## ACL Selector
    ##
    ## Variables:
    ##  - %u: username
    ##  - %c: clientid
    ## auth.mongo.acl_query.selector.3 = clientid=$all

    auth.mongo.acl_query.selector = username=%u

### MongoDB Database

    use mqtt
    db.createCollection("mqtt_user")
    db.createCollection("mqtt_acl")
    db.mqtt_user.ensureIndex({"username":1})

::: tip Tip
The DB name and Collection name are free of choice
:::

### Example of a MongoDB User Collection

    {
        username: "user",
        password: "password hash",
        is_superuser: boolean (true, false),
        created: "datetime"
    }

    db.mqtt_user.insert({username: "test", password: "password hash", is_superuser: false})
    db.mqtt_user:insert({username: "root", is_superuser: true})

### Example of a MongoDB ACL Collection

    {
        username: "username",
        clientid: "clientid",
        publish: ["topic1", "topic2", ...],
        subscribe: ["subtop1", "subtop2", ...],
        pubsub: ["topic/#", "topic1", ...]
    }

    db.mqtt_acl.insert({username: "test", publish: ["t/1", "t/2"], subscribe: ["user/%u", "client/%c"]})
    db.mqtt_acl.insert({username: "admin", pubsub: ["#"]})

### Load Mognodb Auth Plugin

    ./bin/emqx_ctl plugins load emqx_auth_mongo

## JWT Auth Plugin

### Configure JWT Auth

    ## HMAC Hash Secret
    auth.jwt.secret = emqxsecret

    ## From where the JWT string can be got
    auth.jwt.from = password

    ## RSA or ECDSA public key file
    ## auth.jwt.pubkey = etc/certs/jwt_public_key.pem

    ## Enable to verify claims fields
    auth.jwt.verify_claims = off

    ## The checklist of claims to validate
    ##
    ## Variables:
    ##  - %u: username
    ##  - %c: clientid
    # auth.jwt.verify_claims.username = %u

### Load JWT Auth Plugin

    ./bin/emqx_ctl plugins load emqx_auth_jwt
