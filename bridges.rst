
.. _bridge:

========
桥接转发
========

EMQ X 企业版桥接转发 MQTT 消息到 Kafka、RabbitMQ、Pulsar、MQTT Broker 或其他 EMQ X 节点。

桥接插件列表
------------

+-----------------------+--------------------------+---------------------------+
| 存储插件              | 配置文件                 | 说明                      |
+=======================+==========================+===========================+
| emqx_bridge_kafka     | emqx_bridge_kafka.conf   | Kafka 消息存储            |
+-----------------------+--------------------------+---------------------------+
| emqx_bridge_rabbit    | emqx_bridge_rabbit.conf  | RabbitMQ 消息存储         |
+-----------------------+--------------------------+---------------------------+
| emqx_bridge_pulsar    | emqx_bridge_pulsar.conf  | Pulsar 消息存储           |
+-----------------------+--------------------------+---------------------------+
| emqx_bridge_mqtt      | emqx_bridge_mqtt.conf    | MQTT Broker 消息存储      |
+-----------------------+--------------------------+---------------------------+

.. _kafka_bridge:

----------
Kafka 桥接
----------

EMQ X 桥接转发 MQTT 消息到 Kafka 集群:

.. image:: _static/images/bridges_1.png

Kafka 桥接插件配置文件: etc/plugins/emqx_bridge_kafka.conf。

配置 Kafka 集群地址
-------------------

.. code-block:: properties

    ## Kafka Server
    ## bridge.kafka.servers = 127.0.0.1:9092,127.0.0.2:9092,127.0.0.3:9092
    bridge.kafka.servers = 127.0.0.1:9092

    ## Kafka Parition Strategy. option value: per_partition | per_broker
    bridge.kafka.connection_strategy = per_partition

    bridge.kafka.min_metadata_refresh_interval = 5S

    ## Produce writes type. option value: sync | async
    bridge.kafka.produce = sync

    bridge.kafka.produce.sync_timeout = 3S

    ## Base directory for replayq to store messages on disk.
    ## If this config entry if missing or set to undefined,
    ## replayq works in a mem-only manner.
    ## i.e. messages are not queued on disk -- in such case,
    ## the send or send_sync API callers are responsible for
    ## possible message loss in case of application,
    ## network or kafka disturbances. For instance,
    ## in the wolff:send API caller may trap_exit then
    ## react on parition-producer worker pid's 'EXIT'
    ## message to issue a retry after restarting the producer.
    ## bridge.kafka.replayq_dir = /tmp/emqx_bridge_kafka/

    ## default=10MB, replayq segment size.
    ## bridge.kafka.producer.replayq_seg_bytes = 10MB

    ## producer required_acks. option value all_isr | leader_only | none.
    bridge.kafka.producer.required_acks = none

    ## default=10000. Timeout leader wait for replicas before reply to producer.
    bridge.kafka.producer.ack_timeout = 10S

    ## default number of message sets sent on wire before block waiting for acks
    bridge.kafka.producer.max_batch_bytes = 1024KB

    ## by default, send max 1 MB of data in one batch (message set)
    bridge.kafka.producer.min_batch_bytes = 0

    ## Number of batches to be sent ahead without receiving ack for the last request.
    ## Must be 0 if messages must be delivered in strict order.
    bridge.kafka.producer.max_send_ahead = 0

    ## by default, no compression
    ## bridge.kafka.producer.compression = no_compression

    ## by default=base64, option value base64 | plain
    ## bridge.kafka.encode_payload_type = base64

    ## bridge.kafka.sock.buffer = 32KB
    ## bridge.kafka.sock.recbuf = 32KB
    bridge.kafka.sock.sndbuf = 1MB
    ## bridge.kafka.sock.read_packets = 20

配置 Kafka 桥接规则
-------------------

.. code-block:: properties

    ## Bridge Kafka Hooks
    ## ${topic}: the kafka topics to which the messages will be published.
    ## ${filter}: the mqtt topic (may contain wildcard) on which the action will be performed .

    ## Client Connected Record Hook
    bridge.kafka.hook.client.connected.1     = {"topic": "client_connected"}

    ## Client Disconnected Record Hook
    bridge.kafka.hook.client.disconnected.1  = {"topic": "client_disconnected"}

    ## Session Subscribed Record Hook
    bridge.kafka.hook.session.subscribed.1   = {"filter": "#",  "topic": "session_subscribed"}

    ## Session Unsubscribed Record Hook
    bridge.kafka.hook.session.unsubscribed.1 = {"filter": "#",  "topic": "session_unsubscribed"}

    ## Message Publish Record Hook
    bridge.kafka.hook.message.publish.1      = {"filter": "#",  "topic": "message_publish"}

    ## Message Delivered Record Hook
    bridge.kafka.hook.message.delivered.1    = {"filter": "#",  "topic": "message_delivered"}

    ## Message Acked Record Hook
    bridge.kafka.hook.message.acked.1        = {"filter": "#",  "topic": "message_acked"}

    ## More Configures
    ## partitioner strategy:
    ## Option:  random | roundrobin | first_key_dispatch
    ## Example: bridge.kafka.hook.message.publish.1 = {"filter":"#", "topic":"message_publish", "strategy":"random"}

    ## key:
    ## Option: ${clientid} | ${username}
    ## Example: bridge.kafka.hook.message.publish.1 = {"filter":"#", "topic":"message_publish", "key":"${clientid}"}

    ## format:
    ## Option: json | json
    ## Example: bridge.kafka.hook.message.publish.1 = {"filter":"#", "topic":"message_publish", "format":"json"}

Kafka 桥接规则说明
------------------

+-----------------------------------------+------------------+
| 事件                                    | 说明             |
+=========================================+==================+
| bridge.kafka.hook.client.connected.1    | 客户端登录       |
+-----------------------------------------+------------------+
| bridge.kafka.hook.client.disconnected.1 | 客户端退出       |
+-----------------------------------------+------------------+
| bridge.kafka.hook.session.subscribed.1  | 订阅主题         |
+-----------------------------------------+------------------+
| bridge.kafka.hook.session.unsubscribed.1| 取消订阅主题     |
+-----------------------------------------+------------------+
| bridge.kafka.hook.message.publish.1     | 发布消息         |
+-----------------------------------------+------------------+
| bridge.kafka.hook.message.delivered.1   | delivered 消息   |
+-----------------------------------------+------------------+
| bridge.kafka.hook.message.acked.1       | ACK 消息         |
+-----------------------------------------+------------------+

客户端上下线事件转发 Kafka
--------------------------

设备上线 EMQ X 转发上线事件消息到 Kafka:

.. code-block:: javascript

    topic = "client_connected",
    value = {
             "client_id": ${clientid},
             "username": ${username},
             "node": ${node},
             "ts": ${ts}
            }

设备下线 EMQ X 转发下线事件消息到 Kafka:

.. code-block:: javascript

    topic = "client_disconnected",
    value = {
            "client_id": ${clientid},
            "username": ${username},
            "reason": ${reason},
            "node": ${node},
            "ts": ${ts}
            }

客户端订阅主题事件转发 Kafka
----------------------------

.. code-block:: javascript

    topic = session_subscribed

    value = {
             "client_id": ${clientid},
             "topic": ${topic},
             "qos": ${qos},
             "node": ${node},
             "ts": ${timestamp}
            }

客户端取消订阅主题事件转发 Kafka
---------------------------------

.. code-block:: javascript

    topic = session_unsubscribed

    value = {
             "client_id": ${clientid},
             "topic": ${topic},
             "qos": ${qos},
             "node": ${node},
             "ts": ${timestamp}
            }

MQTT 消息转发到 Kafka
---------------------

.. code-block:: javascript

    topic = message_publish

    value = {
             "client_id": ${clientid},
             "username": ${username},
             "topic": ${topic},
             "payload": ${payload},
             "qos": ${qos},
             "node": ${node},
             "ts": ${timestamp}
            }

MQTT 消息派发 (Deliver) 事件转发 Kafka
--------------------------------------

.. code-block:: javascript

    topic = message_delivered

    value = {"client_id": ${clientid},
             "username": ${username},
             "from": ${fromClientId},
             "topic": ${topic},
             "payload": ${payload},
             "qos": ${qos},
             "node": ${node},
             "ts": ${timestamp}
            }

MQTT 消息确认 (Ack) 事件转发 Kafka
-----------------------------------

.. code-block:: javascript

    topic = message_acked

    value = {
             "client_id": ${clientid},
             "username": ${username},
             "from": ${fromClientId},
             "topic": ${topic},
             "payload": ${payload},
             "qos": ${qos},
             "node": ${node},
             "ts": ${timestamp}
            }

Kafka 消费示例
--------------

Kafka 读取 MQTT 客户端上下线事件消息::

    sh kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic client_connected --from-beginning

    sh kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic client_disconnected --from-beginning

Kafka 读取 MQTT 主题订阅事件消息::

    sh kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic session_subscribed --from-beginning

    sh kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic session_unsubscribed --from-beginning

Kafka 读取 MQTT 发布消息::

    sh kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic message_publish --from-beginning

Kafka 读取 MQTT 消息发布 (Deliver)、确认 (Ack) 事件::

    sh kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic message_delivered --from-beginning

    sh kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic message_acked --from-beginning

.. NOTE:: 默认 payload 被 base64 编码，可通过修改配置 bridge.kafka.encode_payload_type 指定 payload 数据格式。

启用 Kafka 桥接插件
-------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_bridge_kafka

.. _rabbit_bridge:

-------------
RabbitMQ 桥接
-------------

EMQ X 桥接转发 MQTT 消息到 RabbitMQ 集群:

.. image:: _static/images/bridges_2.png

RabbitMQ 桥接插件配置文件: etc/plugins/emqx_bridge_rabbit.conf。

配置 RabbitMQ 桥接地址
----------------------

.. code-block:: properties

    ## Rabbit Brokers Server
    bridge.rabbit.1.server = 127.0.0.1:5672

    ## Rabbit Brokers pool_size
    bridge.rabbit.1.pool_size = 4

    ## Rabbit Brokers username
    bridge.rabbit.1.username = guest

    ## Rabbit Brokers password
    bridge.rabbit.1.password = guest

    ## Rabbit Brokers virtual_host
    bridge.rabbit.1.virtual_host = /

    ## Rabbit Brokers heartbeat
    bridge.rabbit.1.heartbeat = 0

    # bridge.rabbit.2.server = 127.0.0.1:5672

    # bridge.rabbit.2.pool_size = 8

    # bridge.rabbit.1.username = guest

    # bridge.rabbit.1.password = guest

    # bridge.rabbit.1.virtual_host = /

    # bridge.rabbit.1.heartbeat = 0

配置 RabbitMQ 桥接规则
----------------------

.. code-block:: properties

    ## Bridge Hooks
    bridge.rabbit.hook.client.subscribe.1 = {"action": "on_client_subscribe", "rabbit": 1, "exchange": "direct:emq.subscription"}

    bridge.rabbit.hook.client.unsubscribe.1 = {"action": "on_client_unsubscribe", "rabbit": 1, "exchange": "direct:emq.unsubscription"}

    bridge.rabbit.hook.message.publish.1 = {"topic": "$SYS/#", "action": "on_message_publish", "rabbit": 1, "exchange": "topic:emq.$sys"}

    bridge.rabbit.hook.message.publish.2 = {"topic": "#", "action": "on_message_publish", "rabbit": 1, "exchange": "topic:emq.pub"}

    bridge.rabbit.hook.message.acked.1 = {"topic": "#", "action": "on_message_acked", "rabbit": 1, "exchange": "topic:emq.acked"}

客户端订阅主题事件转发 RabbitMQ
-------------------------------

.. code-block:: javascript

    routing_key = subscribe
    exchange = emq.subscription
    headers = [{<<"x-emq-client-id">>, binary, ClientId}]
    payload = jsx:encode([{Topic, proplists:get_value(qos, Opts)} || {Topic, Opts} <- TopicTable])

客户端取消订阅事件转发 RabbitMQ
-------------------------------

.. code-block:: javascript

    routing_key = unsubscribe
    exchange = emq.unsubscription
    headers = [{<<"x-emq-client-id">>, binary, ClientId}]
    payload = jsx:encode([Topic || {Topic, _Opts} <- TopicTable]),

MQTT 消息转发 RabbitMQ
----------------------

.. code-block:: javascript

    routing_key = binary:replace(binary:replace(Topic, <<"/">>, <<".">>, [global]),<<"+">>, <<"*">>, [global])
    exchange = emq.$sys | emq.pub
    headers = [{<<"x-emq-publish-qos">>, byte, Qos},
               {<<"x-emq-client-id">>, binary, pub_from(From)},
               {<<"x-emq-publish-msgid">>, binary, emqx_base62:encode(Id)},
               {<<"x-emqx-topic">>, binary, Topic}]
    payload = Payload

MQTT 消息确认 (Ack) 事件转发 RabbitMQ
-------------------------------------

.. code-block:: javascript

    routing_key = puback
    exchange = emq.acked
    headers = [{<<"x-emq-msg-acked">>, binary, ClientId}],
    payload = emqx_base62:encode(Id)

RabbitMQ 订阅消费 MQTT 消息示例
-------------------------------

Python RabbitMQ消费者代码示例:

.. code-block:: javascript

    #!/usr/bin/env python
    import pika
    import sys

    connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))
    channel = connection.channel()

    channel.exchange_declare(exchange='direct:emq.subscription', exchange_type='direct')

    result = channel.queue_declare(exclusive=True)
    queue_name = result.method.queue

    channel.queue_bind(exchange='direct:emq.subscription', queue=queue_name, routing_key= 'subscribe')

    def callback(ch, method, properties, body):
        print(" [x] %r:%r" % (method.routing_key, body))

    channel.basic_consume(callback, queue=queue_name, no_ack=True)

    channel.start_consuming()

其他语言 RabbitMQ 客户端代码示例::

    https://github.com/rabbitmq/rabbitmq-tutorials

启用 RabbitMQ 桥接插件
----------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_bridge_rabbit

.. _pulsar_bridge:

-------------
Pulsar 桥接
-------------

EMQ X 桥接转发 MQTT 消息到 Pulsar 集群:

.. image:: _static/images/bridges_1.png

Pulsar 桥接插件配置文件: etc/plugins/emqx_bridge_pulsar.conf。

配置 Pulsar 集群地址
-------------------

.. code-block:: properties

    ## Cluster support
    ## bridge.pulsar.servers = 127.0.0.1:6650,127.0.0.2:6650,127.0.0.3:6650
    bridge.pulsar.servers = 127.0.0.1:6650

    ## Pick a partition producer and sync/async.
    bridge.pulsar.produce = sync

    ## bridge.pulsar.produce.sync_timeout = 3s

    ## bridge.pulsar.producer.batch_size = 1000

    ## by default, no compression
    ## bridge.pulsar.producer.compression = no_compression

    ## base64 | plain
    ## bridge.pulsar.encode_payload_type = base64

    ## bridge.pulsar.sock.buffer = 32KB
    ## bridge.pulsar.sock.recbuf = 32KB
    bridge.pulsar.sock.sndbuf = 1MB
    ## bridge.pulsar.sock.read_packets = 20

配置 Pulsar 桥接规则
---------------------

.. code-block:: properties

    ## Bridge Pulsar Hooks
    ## ${topic}: the pulsar topics to which the messages will be published.
    ## ${filter}: the mqtt topic (may contain wildcard) on which the action will be performed .

    ## Client Connected Record Hook
    bridge.pulsar.hook.client.connected.1     = {"topic": "client_connected"}

    ## Client Disconnected Record Hook
    bridge.pulsar.hook.client.disconnected.1  = {"topic": "client_disconnected"}

    ## Session Subscribed Record Hook
    bridge.pulsar.hook.session.subscribed.1   = {"filter": "#",  "topic": "session_subscribed"}

    ## Session Unsubscribed Record Hook
    bridge.pulsar.hook.session.unsubscribed.1 = {"filter": "#",  "topic": "session_unsubscribed"}

    ## Message Publish Record Hook
    bridge.pulsar.hook.message.publish.1      = {"filter": "#",  "topic": "message_publish"}

    ## Message Delivered Record Hook
    bridge.pulsar.hook.message.delivered.1    = {"filter": "#",  "topic": "message_delivered"}

    ## Message Acked Record Hook
    bridge.pulsar.hook.message.acked.1        = {"filter": "#",  "topic": "message_acked"}

    ## More Configures
    ## partitioner strategy:
    ## Option:  random | roundrobin | first_key_dispatch
    ## Example: bridge.pulsar.hook.message.publish.1 = {"filter":"#", "topic":"message_publish", "strategy":"random"}

    ## key:
    ## Option: ${clientid} | ${username}
    ## Example: bridge.pulsar.hook.message.publish.1 = {"filter":"#", "topic":"message_publish", "key":"${clientid}"}

    ## format:
    ## Option: json | json
    ## Example: bridge.pulsar.hook.message.publish.1 = {"filter":"#", "topic":"message_publish", "format":"json"}

Pulsar 桥接规则说明
-------------------

+-----------------------------------------+------------------+
| 事件                                    | 说明             |
+=========================================+==================+
| bridge.pulsar.hook.client.connected.1    | 客户端登录      |
+-----------------------------------------+------------------+
| bridge.pulsar.hook.client.disconnected.1 | 客户端退出      |
+-----------------------------------------+------------------+
| bridge.pulsar.hook.session.subscribed.1  | 订阅主题        |
+-----------------------------------------+------------------+
| bridge.pulsar.hook.session.unsubscribed.1| 取消订阅主题    |
+-----------------------------------------+------------------+
| bridge.pulsar.hook.message.publish.1     | 发布消息        |
+-----------------------------------------+------------------+
| bridge.pulsar.hook.message.delivered.1   | delivered 消息  |
+-----------------------------------------+------------------+
| bridge.pulsar.hook.message.acked.1       | ACK 消息        |
+-----------------------------------------+------------------+

客户端上下线事件转发 Pulsar
----------------------------

设备上线 EMQ X 转发上线事件消息到 Pulsar:

.. code-block:: javascript

    topic = "client_connected",
    value = {
             "client_id": ${clientid},
             "username": ${username},
             "node": ${node},
             "ts": ${ts}
            }

设备下线 EMQ X 转发下线事件消息到 Pulsar:

.. code-block:: javascript

    topic = "client_disconnected",
    value = {
            "client_id": ${clientid},
            "username": ${username},
            "reason": ${reason},
            "node": ${node},
            "ts": ${ts}
            }

客户端订阅主题事件转发 Pulsar
------------------------------

.. code-block:: javascript

    topic = session_subscribed

    value = {
             "client_id": ${clientid},
             "topic": ${topic},
             "qos": ${qos},
             "node": ${node},
             "ts": ${timestamp}
            }

客户端取消订阅主题事件转发 Pulsar
---------------------------------

.. code-block:: javascript

    topic = session_unsubscribed

    value = {
             "client_id": ${clientid},
             "topic": ${topic},
             "qos": ${qos},
             "node": ${node},
             "ts": ${timestamp}
            }

MQTT 消息转发到 Pulsar
-----------------------

.. code-block:: javascript

    topic = message_publish

    value = {
             "client_id": ${clientid},
             "username": ${username},
             "topic": ${topic},
             "payload": ${payload},
             "qos": ${qos},
             "node": ${node},
             "ts": ${timestamp}
            }

MQTT 消息派发 (Deliver) 事件转发 Pulsar
---------------------------------------

.. code-block:: javascript

    topic = message_delivered

    value = {"client_id": ${clientid},
             "username": ${username},
             "from": ${fromClientId},
             "topic": ${topic},
             "payload": ${payload},
             "qos": ${qos},
             "node": ${node},
             "ts": ${timestamp}
            }

MQTT 消息确认 (Ack) 事件转发 Pulsar
-----------------------------------

.. code-block:: javascript

    topic = message_acked

    value = {
             "client_id": ${clientid},
             "username": ${username},
             "from": ${fromClientId},
             "topic": ${topic},
             "payload": ${payload},
             "qos": ${qos},
             "node": ${node},
             "ts": ${timestamp}
            }

Pulsar 消费示例
----------------

Pulsar 读取 MQTT 客户端上下线事件消息::

    sh pulsar-client consume client_connected  -s "client_connected" -n 1000

    sh pulsar-client consume client_disconnected  -s "client_disconnected" -n 1000

Pulsar 读取 MQTT 主题订阅事件消息::

    sh pulsar-client consume session_subscribed  -s "session_subscribed" -n 1000

    sh pulsar-client consume session_unsubscribed  -s "session_unsubscribed" -n 1000

Pulsar 读取 MQTT 发布消息::

    sh pulsar-client consume message_publish  -s "message_publish" -n 1000

Pulsar 读取 MQTT 消息发布 (Deliver)、确认 (Ack) 事件::

    sh pulsar-client consume message_delivered  -s "message_delivered" -n 1000

    sh pulsar-client consume message_acked  -s "message_acked" -n 1000

.. NOTE:: 默认 payload 被 base64 编码，可通过修改配置 bridge.pulsar.encode_payload_type 指定 payload 数据格式。

启用 Pulsar 桥接插件
---------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_bridge_pulsar

