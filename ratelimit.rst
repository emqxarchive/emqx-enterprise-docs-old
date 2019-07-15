
.. _ratelimit:

=====================
速率限制 (Rate Limit)
=====================

EMQ X 企业版支持多种限速方式，以保证系统可靠稳定运行。

-------------------------------
并发连接数量 - max_connections
-------------------------------

MQTT TCP 或 SSL 监听器，配置允许的最大并发连接数：

.. code-block:: properties

    ## 最大并发连接数; 针对该 <name> 监听的端口有效
    listener.tcp.<name>.max_connections = 102400

    listener.ssl.<name>.max_connections = 102400

-----------------------------
连接速率限制 - max_conn_rate
-----------------------------

MQTT TCP 或 SSL 监听器，配置最大允许连接速率：

.. code-block:: properties

    ## 每秒允许的最大连接数; 针对该 <name> 监听的端口有效
    listener.tcp.<name>.max_conn_rate = 1000

    listener.ssl.<name>.max_conn_rate = 1000

--------------------------
连接流量限制 - rate_limit
--------------------------

MQTT TCP 或 SSL 监听器，设置单个连接流量限制：

.. code-block:: properties

    ## 单个连接流量限制
    ##
    ## 格式: rate,burst
    ##   - rate: 限制的平均速率值
    ##   - burst: 单次检测最大允许的流量值; 为避免频繁的被流控限制, 该值
    ##            建议为 `(max_packet_size * active_n)/2`
    ##            (`max_packet_size` 和 `active_n` 为 `etc/emqx.conf` 
    ##            中的另外俩个配置项
    ## 单位: Byte
    ## listener.tcp.<name>.rate_limit = 1024,52428800

    ## listener.ssl.<name>.rate_limit = 1024,52428800

-------------------------------
发布速率限制 - max_publish_rate
-------------------------------

可以为每个 `<name>` 设置消息发布速率限制：

.. code-block:: properties

    ## 最大 MQTT 消息发布速率限制
    ##
    ## 例: 每分钟 10 条
    ## zone.<name>.publish_limit = 10,1m

