
.. _ratelimit:

=====================
速率限制 (Rate Limit)
=====================

EMQ X 企业版支持多种限速方式，以保证系统可靠稳定运行。

--------------------------
并发连接数量 - max_clients
--------------------------

MQTT TCP 或 SSL 监听器，配置最大允许并发连接数：

.. code-block:: properties

    ## Maximum number of concurrent MQTT/TCP connections.
    ##
    ## Value: Number
    listener.tcp.<name>.max_clients = 102400

    ## Maximum number of concurrent MQTT/SSL connections.
    ##
    ## Value: Number
    listener.ssl.<name>.max_clients = 102400

----------------------------
连接速率限制 - max_conn_rate
----------------------------

MQTT TCP 或 SSL 监听器，配置最大允许连接速率，默认每秒1000连接：

.. code-block:: properties

    ## Maximum external connections per second.
    ##
    ## Value: Number
    listener.tcp.<name>.max_conn_rate = 1000

    ## Maximum MQTT/SSL connections per second.
    ##
    ## Value: Number
    listener.ssl.<name>.max_conn_rate = 1000

-------------------------
连接流量限制 - rate_limit
-------------------------

MQTT TCP 或 SSL 监听器，设置单个连接流量限制：

.. code-block:: properties

    ## Rate limit for the external MQTT/TCP connections. Format is 'rate,burst'.
    ##
    ## Value: rate,burst
    ## Unit: Bps
    ## listener.tcp.<name>.rate_limit = 1024,4096

    ## Rate limit for the external MQTT/SSL connections.
    ##
    ## Value: rate,burst
    ## Unit: Bps
    ## listener.ssl.<name>.rate_limit = 1024,4096

-------------------------------
发布速率限制 - max_publish_rate
-------------------------------

MQTT TCP 或 SSL 监听器，设置单个连接发布消息速率限制：

.. code-block:: properties

    ## Maximum publish rate of MQTT messages.
    ##
    ## Value: Number,Seconds
    ## Default: 10 messages per minute
    ## listener.tcp.<name>.max_publish_rate = 10,60

    ## Maximum publish rate of MQTT messages.
    ##
    ## See: listener.tcp.<name>.max_publish_rate
    ##
    ## Value: Number,Seconds
    ## Default: 10 messages per minute
    ## listener.ssl.external.max_publish_rate = 10,60


