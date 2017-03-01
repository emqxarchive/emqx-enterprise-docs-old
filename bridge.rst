
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
