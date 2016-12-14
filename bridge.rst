
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

