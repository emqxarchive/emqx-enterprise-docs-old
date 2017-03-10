
.. _bridge:

========
节点桥接
========

.. _bridge_emq:

-------------
EMQ节点间桥接
-------------

*EMQ* 消息服务器支持多节点桥接模式互联::

                  ---------                     ---------                     ---------
    Publisher --> | Node1 | --Bridge Forward--> | Node2 | --Bridge Forward--> | Node3 | --> Subscriber
                  ---------                     ---------                     ---------

节点间桥接与集群不同，不复制主题树与路由表，只按桥接规则转发MQTT消息。

EMQ节点桥接配置
---------------

假设在本机创建两个EMQ节点，并创建一条桥接转发全部传感器(sensor)主题消息:

+---------+------------------+----------+
| 目录    | 节点             | MQTT端口 |
+---------+------------------+----------+
| emqttd1 | emq1@127.0.0.1   | 1883     |
+---------+------------------+----------+
| emqttd2 | emq2@127.0.0.1   | 2883     |
+---------+------------------+----------+

启动emq1, emq2节点:

.. code-block:: bash

    cd emqttd1/ && ./bin/emqttd start
    cd emqttd2/ && ./bin/emqttd start

emq1节点上创建到emq2桥接:

.. code-block:: bash

    $ ./bin/emqctl bridges start emq2@127.0.0.1 sensor/#

    bridge is started.

    $ ./bin/emqctl bridges list

    bridge: emq1@127.0.0.1--sensor/#-->emq2@127.0.0.1

测试emq1--sensor/#-->emq2的桥接:

.. code-block:: bash

    #emqttd2节点上

    mosquitto_sub -t sensor/# -p 2883 -d

    #emqttd1节点上

    mosquitto_pub -t sensor/1/temperature -m "37.5" -d

删除桥接:

.. code-block:: bash

    ./bin/emqctl bridges stop emq2@127.0.0.1 sensor/#

.. _bridge_mosquitto:

-------------
mosquitto桥接
-------------

mosquitto可以普通MQTT连接方式，桥接到 *EMQ* 消息服务器::

                 -------------             -----------------
    Sensor ----> | mosquitto | --Bridge--> |               |
                 -------------             |      EMQ      |
                 -------------             |    Cluster    |
    Sensor ----> | mosquitto | --Bridge--> |               |
                 -------------             -----------------

mosquitto.conf
--------------

本机2883端口启动emqttd消息服务器，1883端口启动mosquitto并创建桥接。

mosquitto.conf配置::

    connection emqttd
    address 127.0.0.1:2883
    topic sensor/# out 2

    # Set the version of the MQTT protocol to use with for this bridge. Can be one
    # of mqttv31 or mqttv311. Defaults to mqttv31.
    bridge_protocol_version mqttv311

.. _bridge_rsmb:

--------
rsmb桥接
--------

本机2883端口启动emqttd消息服务器，1883端口启动rsmb并创建桥接。

broker.cfg桥接配置::

    connection emqttd
    addresses 127.0.0.1:2883
    topic sensor/#

.. _bridge_kafka:

------------
Kafka消息桥接
------------
*EMQX* 消息服务器支持消息桥接到Kafka server::

                   ---------                      --------- 
    Publisher -->  |  EMQX  | --Bridge Forward--> | Kafka  |  --> Subscriber
                   ---------                      --------- 

配置Kafka消息桥接
-----------------------

etc/plugins/emqx_bridge_kafka.conf:

.. code-block:: properties

    ## Kafka Server
    bridge.kafka.pool1.server = 127.0.0.1:9092

    ## Kafka Pool Size 
    bridge.kafka.pool1.pool_size = 8
    
    ## Kafka Parition Strategy
    bridge.kafka.parition_strategy = random
    
    ## Client Connected Record Hook
    bridge.kafka.hook.client.connected.1 = {"action": "on_client_connected", "pool": "pool1", "topic": "client_connected"}

    ## Client Disconnected Record Hook
    bridge.kafka.hook.client.disconnected.1 = {"action": "on_client_disconnected", "pool": "pool1", "topic": "client_disconnected"}

    ## Session Subscribed Record Hook
    bridge.kafka.hook.session.subscribed.1 = {"action": "on_session_subscribed", "filter": "#", "pool": "pool1", "topic": "session_subscribed"}

    ## Session Unsubscribed Record Hook
    bridge.kafka.hook.session.unsubscribed.1 = {"action": "on_session_unsubscribed", "filter": "#", "pool": "pool1", "topic": "session_unsubscribed"}

    ## Message Publish Record Hook
    bridge.kafka.hook.message.publish.1 = {"action": "on_message_publish", "filter": "#", "pool": "pool1", "topic": "message_publish"}

    ## Message Delivered Record Hook
    bridge.kafka.hook.message.delivered.1 = {"action": "on_message_delivered", "filter": "#", "pool": "pool1", "topic": "message_delivered"}

    ## Message Acked Record Hook
    bridge.kafka.hook.message.acked.1 = {"action": "on_message_acked", "filter": "#", "pool": "pool1", "topic": "message_acked"}

*bridge* 消息桥接规则包括:

+------------------------+----------------------------------+
| action                 | 说明                             |
+========================+==================================+
| on_client_connected    | 客户端登录                       |
+------------------------+----------------------------------+
| on_client_disconnected | 客户端退出                       |
+------------------------+----------------------------------+
| on_session_subscribed  | 订阅主题                         |
+------------------------+----------------------------------+
| on_session_unsubscribed| 取消订阅主题                     |
+------------------------+----------------------------------+
| on_message_publish     | 发布消息                         |
+------------------------+----------------------------------+
| on_message_delivered   | delivered消息                    |
+------------------------+----------------------------------+
| on_message_acked       | ACK消息                          |
+------------------------+----------------------------------+

加载Kafka消息桥接插件
-------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_bridge_kafka

Kafka EMQ客户端连接消息(topic, json)
---------------------------------

.. code-block:: javascript
    
    topic = "client_connected",
    value = {"client_id": ClientId, 
             "node": node(), 
             "ts": emqx_time:now_secs()}

Kafka EMQ客户端断开连接消息(topic, json)
---------------------------------

.. code-block:: javascript
    
    topic = "client_disconnected",
    value = {"client_id": ClientId, 
     "reason": Reason, 
     "node": node(), 
     "ts": emqx_time:now_secs()}

Kafka EMQ订阅主题消息(topic, json)
---------------------------------

.. code-block:: javascript
    
    topic = session_subscribed
    value = {"client_id": ClientId, 
     "topic": Topic, 
     "qos": Qos,
     "node": node(), 
     "ts": emqx_time:now_secs()}

Kafka EMQ取消订阅主题消息(topic, json)
---------------------------------

.. code-block:: javascript
    
    topic = session_unsubscribed
    value = {"client_id": ClientId, 
             "topic": Topic, 
             "qos": Qos,
             "node": node(), 
             "ts": emqx_time:now_secs()}

Kafka EMQ发布消息(topic, json)
---------------------------------

.. code-block:: javascript

    topic = message_publish
    value = {"client_id": ClientId, 
             "username": Username, 
             "topic": Topic, 
             "payload": Payload, 
             "qos": Qos,
             "node": node(), 
             "ts": emqx_time:now_secs()}

Kafka EMQ Delivered消息(topic, json)
---------------------------------

.. code-block:: javascript
    
    topic = message_delivered
    value = {"client_id": ClientId, 
             "username": Username, 
             "from": FromClientId,
             "topic": Topic, 
             "payload": Payload, 
             "qos": Qos,
             "node": node(), 
             "ts": emqx_time:now_secs()}


Kafka EMQ Acked消息(json)
---------------------------------

.. code-block:: javascript
    
    topic = message_acked
    value = {"client_id": ClientId, 
             "username": Username,
             "from": FromClientId, 
             "topic": Topic, 
             "payload": Payload, 
             "qos": Qos,
             "node": node(), 
             "ts": emqx_time:now_secs()}
     
示例
----
    
Kafka消费者订 emq阅客户端连接消息::

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic client_connected --from-beginning
    
Kafka消费者订 emq阅客户端断开连接消息::

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic client_disconnected --from-beginning
    
Kafka消费者订阅 emq订阅主题消息消息::

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic session_subscribed --from-beginning
    
Kafka消费者订阅 emq取消订阅主题消息消息::

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic session_unsubscribed --from-beginning
    
Kafka消费者订阅 emq发布消息::

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic message_publish --from-beginning
    
Kafka消费者订阅 emq delivered消息::

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic message_delivered --from-beginning
    
Kafka消费者订阅 emq acked消息::

    sh kafka-console-consumer.sh --zookeeper localhost:2181 --topic message_acked --from-beginning
    
注意:: 

  payload被base64编码，因此kafka消费者应该做base64解码以获得原始的payload。


.. _bridge_rabbit:

---------------
RabbitMQ消息桥接
---------------
*EMQX* 消息服务器支持消息桥接到RabbitMQ server::

                   ---------                      ------------ 
    Publisher -->  |  EMQX  | --Bridge Forward--> | RabbitMQ |  --> Subscriber
                   ---------                      ------------ 

配置RabbitMQ消息桥接
-----------------------

etc/plugins/emqx_bridge_rabbit.conf:

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

    ## Bridge Hooks
    bridge.rabbit.hook.client.subscribe.1 = {"action": "on_client_subscribe", "rabbit": 1, "exchange": "direct:emq.subscription"}

    bridge.rabbit.hook.client.unsubscribe.1 = {"action": "on_client_unsubscribe", "rabbit": 1, "exchange": "direct:emq.unsubscription"}

    bridge.rabbit.hook.message.publish.1 = {"topic": "$SYS/#", "action": "on_message_publish", "rabbit": 1, "exchange": "topic:emq.$sys"}

    bridge.rabbit.hook.message.publish.2 = {"topic": "#", "action": "on_message_publish", "rabbit": 1, "exchange": "topic:emq.pub"}

    bridge.rabbit.hook.message.acked.1 = {"action": "on_message_acked", "rabbit": 1, "exchange": "topic:emq.acked"}

加载RabbitMQ消息桥接插件
----------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_bridge_rabbit

RabbitMQ EMQX客户端订阅主题(exchange, routing_key, headers, payload)
-----------------------------------------------------

.. code-block:: javascript

    routing_key = subscribe
    exchange = emq.subscription
    headers = [{<<"x-emq-client-id">>, binary, ClientId}]
    payload = jsx:encode([{Topic, proplists:get_value(qos, Opts)} || {Topic, Opts} <- TopicTable])

RabbitMQ EMQX客户端取消订阅主题(exchange, routing_key, headers, payload)
-----------------------------------------------------

.. code-block:: javascript

    routing_key = unsubscribe
    exchange = emq.unsubscription
    headers = [{<<"x-emq-client-id">>, binary, ClientId}]
    payload = jsx:encode([Topic || {Topic, _Opts} <- TopicTable]),


RabbitMQ EMQX客户端发布消息(exchange, routing_key, headers, payload)
-----------------------------------------------------

.. code-block:: javascript

    routing_key = binary:replace(binary:replace(Topic, <<"/">>, <<".">>, [global]),<<"+">>, <<"*">>, [global])
    exchange = emq.$sys | emq.pub
    headers = [{<<"x-emq-publish-qos">>, byte, Qos},
               {<<"x-emq-client-id">>, binary, pub_from(From)},
               {<<"x-emq-publish-msgid">>, binary, emqx_base62:encode(Id)}]
    payload = Payload

RabbitMQ EMQX客户端发布ACK消息(exchange, routing_key, headers, payload)
-----------------------------------------------------

.. code-block:: javascript

    routing_key = puback
    exchange = emq.acked
    headers = [{<<"x-emq-msg-acked">>, binary, ClientId}],
    payload = emqx_base62:encode(Id)

示例
----

python RabbitMQ消费者代码示例::

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
    

其他语言RabbitMQ消费者代码示例请查看::

    https://github.com/rabbitmq/rabbitmq-tutorials

