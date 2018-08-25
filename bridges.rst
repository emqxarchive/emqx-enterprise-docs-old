
.. _bridge:

========
桥接转发
========

EMQ X 企业版桥接转发MQTT消息到Kafka、RabbitMQ或其他EMQ X节点。同时支持mosquitto、rsmb以普通MQTT连接方式桥接到EMQ X。

.. _kafka_bridge:

---------
Kafka桥接
---------

EMQ X 桥接转发MQTT消息到Kafka集群:

.. image:: _static/images/bridges_1.png

Kafka桥接插件配置文件: etc/plugins/emqx_bridge_kafka.conf。

配置Kafka集群地址
-----------------

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

    ## bridge.kafka.encode_payload_type = base64

    ## bridge.kafka.sock.buffer = 32KB
    ## bridge.kafka.sock.recbuf = 32KB
    bridge.kafka.sock.sndbuf = 1MB
    ## bridge.kafka.sock.read_packets = 20

配置Kafka桥接规则
-----------------

.. code-block:: properties

    ## Bridge Kafka Hooks
    ## ${topic}: the kafka topics to which the messages will be published.
    ## ${filter}: the mqtt topic (may contain wildcard) on which the action will be performed .

    bridge.kafka.hook.client.connected.1     = {"topic": "client_connected"}
    bridge.kafka.hook.client.disconnected.1  = {"topic": "client_disconnected"}
    bridge.kafka.hook.session.subscribed.1   = {"filter": "#",  "topic": "session_subscribed"}
    bridge.kafka.hook.session.unsubscribed.1 = {"filter": "#",  "topic": "session_unsubscribed"}
    bridge.kafka.hook.message.publish.1      = {"filter": "#",  "topic": "message_publish"}
    bridge.kafka.hook.message.delivered.1    = {"filter": "#",  "topic": "message_delivered"}
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

Kafka桥接规则说明
-----------------

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
| bridge.kafka.hook.message.delivered.1   | delivered消息    |
+-----------------------------------------+------------------+
| bridge.kafka.hook.message.acked.1       | ACK消息          |
+-----------------------------------------+------------------+

客户端上下线事件转发Kafka
-------------------------

设备上线 EMQ X 转发上线事件消息到Kafka:

.. code-block:: javascript

    topic = "client_connected",
    value = {
             "client_id": ${clientid},
             "node": ${node},
             "ts": ${ts}
            }

设备下线 EMQ X 转发下线事件消息到Kafka:

.. code-block:: javascript

    topic = "client_disconnected",
    value = {
            "client_id": ${clientid},
            "reason": ${reason},
            "node": ${node},
            "ts": ${ts}
            }

客户端订阅主题事件转发Kafka
---------------------------

.. code-block:: javascript

    topic = session_subscribed

    value = {
             "client_id": ${clientid},
             "topic": ${topic},
             "qos": ${qos},
             "node": ${node},
             "ts": ${timestamp}
            }

客户端取消订阅主题事件转发Kafka
--------------------------------

.. code-block:: javascript

    topic = session_unsubscribed

    value = {
             "client_id": ${clientid},
             "topic": ${topic},
             "qos": ${qos},
             "node": ${node},
             "ts": ${timestamp}
            }

MQTT消息转发到Kafka
-------------------

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

MQTT消息派发(Deliver)事件转发Kafka
----------------------------------

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

MQTT消息确认(Ack)事件转发Kafka
-------------------------------

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

Kafka消费示例
-------------

Kafka读取MQTT客户端上下线事件消息::

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic client_connected --from-beginning

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic client_disconnected --from-beginning

Kafka读取MQTT主题订阅事件消息::

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic session_subscribed --from-beginning

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic session_unsubscribed --from-beginning

Kafka读取MQTT发布消息::

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic message_publish --from-beginning

Kafka读取MQTT消息发布(Deliver)、确认(Ack)事件::

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic message_delivered --from-beginning

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic message_acked --from-beginning

.. NOTE:: payload为base64编码

启用Kafka桥接插件
-----------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_bridge_kafka

.. _rabbit_bridge:

------------
RabbitMQ桥接
------------

EMQ X 桥接转发 MQTT 消息到 RabbitMQ 集群:

.. image:: _static/images/bridges_2.png

RabbitMQ桥接插件配置文件: etc/plugins/emqx_bridge_rabbit.conf。

配置RabbitMQ桥接地址
--------------------

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
    bridge.rabbit.1.heartbeat = 30

    # bridge.rabbit.2.server = 127.0.0.1:5672

    # bridge.rabbit.2.pool_size = 8

    # bridge.rabbit.2.username = guest

    # bridge.rabbit.2.password = guest

    # bridge.rabbit.2.virtual_host = /

    # bridge.rabbit.2.heartbeat = 30

配置RabbitMQ桥接规则
--------------------

.. code-block:: properties

    ## Bridge Hooks
    bridge.rabbit.hook.client.subscribe.1 = {"action": "on_client_subscribe", "rabbit": 1, "exchange": "direct:emq.subscription"}

    bridge.rabbit.hook.client.unsubscribe.1 = {"action": "on_client_unsubscribe", "rabbit": 1, "exchange": "direct:emq.unsubscription"}

    bridge.rabbit.hook.message.publish.1 = {"topic": "$SYS/#", "action": "on_message_publish", "rabbit": 1, "exchange": "topic:emq.$sys"}

    bridge.rabbit.hook.message.publish.2 = {"topic": "#", "action": "on_message_publish", "rabbit": 1, "exchange": "topic:emq.pub"}

    bridge.rabbit.hook.message.acked.1 = {"topic": "#", "action": "on_message_acked", "rabbit": 1, "exchange": "topic:emq.acked"}

客户端订阅主题事件转发RabbitMQ
------------------------------

.. code-block:: javascript

    routing_key = subscribe
    exchange = emq.subscription
    headers = [{<<"x-emq-client-id">>, binary, ClientId}]
    payload = jsx:encode([{Topic, proplists:get_value(qos, Opts)} || {Topic, Opts} <- TopicTable])

客户端取消订阅事件转发RabbitMQ
------------------------------

.. code-block:: javascript

    routing_key = unsubscribe
    exchange = emq.unsubscription
    headers = [{<<"x-emq-client-id">>, binary, ClientId}]
    payload = jsx:encode([Topic || {Topic, _Opts} <- TopicTable]),

MQTT消息转发RabbitMQ
--------------------

.. code-block:: javascript

    routing_key = binary:replace(binary:replace(Topic, <<"/">>, <<".">>, [global]),<<"+">>, <<"*">>, [global])
    exchange = emq.$sys | emq.pub
    headers = [{<<"x-emq-publish-qos">>, byte, Qos},
               {<<"x-emq-client-id">>, binary, pub_from(From)},
               {<<"x-emq-publish-msgid">>, binary, emqx_base62:encode(Id)}]
    payload = Payload

MQTT消息确认(Ack)事件转发RabbitMQ
---------------------------------

.. code-block:: javascript

    routing_key = puback
    exchange = emq.acked
    headers = [{<<"x-emq-msg-acked">>, binary, ClientId}],
    payload = emqx_base62:encode(Id)

RabbitMQ订阅消费MQTT消息示例
----------------------------

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

其他语言RabbitMQ客户端代码示例::

    https://github.com/rabbitmq/rabbitmq-tutorials

启用RabbitMQ桥接插件
--------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_bridge_rabbit

.. _emqx_bridge:

----------
EMQ X 桥接
----------

EMQ X 支持多节点间桥接模式互联:

.. image:: _static/images/bridges_3.png

假设创建emqx1, emqx2两个节点:

+---------+--------------------+
| 目录    | 节点               |
+---------+--------------------+
| emqx1   | emqx1@192.168.1.10 |
+---------+--------------------+
| emqx2   | emqx2@192.168.1.20 |
+---------+--------------------+

启用emqx1, emqx2节点后，emqx1节点创建到emqx2桥接，转发全部'sensor/#'主题消息到emqx2:

.. code-block:: bash

    $ ./bin/emqx_ctl bridges start emqx2@192.168.1.20 sensor/#

    bridge is started.

    $ ./bin/emqx_ctl bridges list

    bridge: emqx1@127.0.0.1--sensor/#-->emqx2@127.0.0.1

测试emqx1--sensor/#-->emqx2的桥接:

.. code-block:: bash

    #emqx2节点上

    mosquitto_sub -t sensor/# -p 2883 -d

    #emqx1节点上

    mosquitto_pub -t sensor/1/temperature -m "37.5" -d

删除桥接:

.. code-block:: bash

    ./bin/emqx_ctl bridges stop emqx2@127.0.0.1 sensor/#

.. _mosquitto_bridge:

-------------
mosquitto桥接
-------------

mosquitto 可以普通 MQTT 连接方式，桥接到EMQ X 服务器集群:

.. image:: _static/images/bridges_4.png

mosquitto.conf桥接配置示例::

    connection emqx
    address 192.168.0.10:1883
    topic sensor/# out 2

    # Set the version of the MQTT protocol to use with for this bridge. Can be one
    # of mqttv31 or mqttv311. Defaults to mqttv31.
    bridge_protocol_version mqttv311

.. _rsmb_bridge:

--------
rsmb桥接
--------

rsmb以普通MQTT连接方式，桥接到 EMQ X 服务器集群。

rsmb broker.cfg示例配置::

    connection emqx
    addresses 127.0.0.1:2883
    topic sensor/#

