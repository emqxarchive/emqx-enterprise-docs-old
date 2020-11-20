# Protocol

## MQTT Protocol

### Summary

MQTT is a machine-to-machine (M2M)/"Internet of Things" connectivity protocol. It was designed as an extremely lightweight publish/subscribe messaging transport.

It is useful for connections with remote locations where a small code footprint is required and/or network bandwidth is at a premium.

Official Website: [ http://mqtt.org ](http://mqtt.org)

MQTT V3.1.1 Protocol: [ http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html ](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html)

### Features

1. OASIS Standard protocol, easy to implement
2. Pub/Sub messaging transport, supported One-to-More transport
3. Over TCP/IP
4. Lightweight and Compact packet structure; Only 1 byte fixed header and 2 bytes keepalive frame
5. Qos supported, guarantee the reach of messages

### Application Scenes

The MQTT protocol is widely used in the fields of Internet of Things, Mobile Internet, Intelligent Hardware, Vehicle Networking, and Power Energy.

1. M2M Communication, Data Collection for IoT
2. Message Push Services for mobile device, and Web
3. Instant message. e.g.: Facebook Messenger
4. Smart Home, Intelligent Hardware
5. Vehicle Networking Communication, Data collection for pile of electric station
6. Smart city, Telemedicine, Distance Education
7. Electricity, oil and energy industry market

### Message Routing

The MQTT message routing rule based on Topic, which is similar to URL path, such as:

    chat/room/1

    sensor/10/temperature

    sensor/+/temperature

    $SYS/broker/metrics/packets/received

    $SYS/broker/metrics/#

The topic level separator is used to introduce structure into the Topic Name. If present, it divides the Topic Name into multiple "topic levels"

A subscription’s Topic Filter can contain '+', '#' wildcard characters, which allow you to subscribe to multiple topics at once:

    '+': Match only one topic level; e.g.: a/+, match a/x, a/y

    '#': Match any number of levels; e.g: a/#, match a/x, a/b/c/d

Publisher and Subscriber exchanges messages by message routing with topic; e.g.: Pub/Sub message by `mosquitto` :

    mosquitto_sub -t a/b/+ -q 1

    mosquitto_pub -t a/b/c -m hello -q 1

::: tip Tip
Topic wildcards are only used for subscribing to topics, not publishing
:::

### MQTT V3.1.1 Protocol

#### Packet Format

#### Fixed Header

    +----------+-----+-----+-----+-----+-----+-----+-----+-----+
    | Bit      |  7  |  6  |  5  |  4  |  3  |  2  |  1  |  0  |
    +----------+-----+-----+-----+-----+-----+-----+-----+-----+
    | byte1    |   MQTT Packet type    |         Flags         |
    +----------+-----------------------+-----------------------+
    | byte2... |   Remaining Length                            |
    +----------+-----------------------------------------------+

#### Packet Type

| Type Name   | Value | Description                         |
| ----------- | ----- | ----------------------------------- |
| CONNECT     | 1     | Client request to connect to Server |
| CONNACK     | 2     | Connect acknowledgment              |
| PUBLISH     | 3     | Publish message                     |
| PUBACK      | 4     | Publish acknowledgment              |
| PUBREC      | 5     | Publish received                    |
| PUBREL      | 6     | Publish release                     |
| PUBCOMP     | 7     | Publish complete                    |
| SUBSCRIBE   | 8     | Client subscribe request            |
| SUBACK      | 9     | Subscribe acknowledgment            |
| UNSUBSCRIBE | 10    | Unsubscribe request                 |
| UNSUBACK    | 11    | Unsubscribe acknowledgment          |
| PINGREQ     | 12    | PING request                        |
| PINGRESP    | 13    | PING response                       |
| DISCONNECT  | 14    | Client is disconnecting             |

#### PUBLISH

PUBLISH packet is sent from a Client to a Server or from Server to a Client to transport an Application Message

PUBACK packet is the response to a PUBLISH Packet with QoS1

PUBREC/PUBREL/PUBCOMP is a serial of packets for Qos2 exchange flow

#### PINGREQ/PINGRESP

PINGREQ/PINGRESP packets are used in Keep Alive processing.

The PINGREQ packet is sent from a Client to the Server. It can be used to:

1. Indicate to the Server that the Client is alive in the absence of any other Control Packets being sent from the Client to the Server.
2. Request that the Server responds to confirm that it is alive.
3. Exercise the network to indicate that the Network Connection is active.

A PINGRESP Packet is sent by the Server to the Client in response to a PINGREQ Packet. It indicates that the Server is alive.

### QoS (Quality of Service)

The message QoS guarantees are not end-to-end, but between client and server.

The QoS level at which subscribers receive MQTT messages ultimately depends on the QoS of published messages and topic subscriptions.

| Message QoS | Subscribed Qos | Received QoS |
| ----------- | -------------- | ------------ |
| 0           | 0              | 0            |
| 0           | 1              | 0            |
| 0           | 2              | 0            |
| 1           | 0              | 0            |
| 1           | 1              | 1            |
| 1           | 2              | 1            |
| 2           | 0              | 0            |
| 2           | 1              | 1            |
| 2           | 2              | 2            |

#### Message Flow - Qos0

![image](./_static/images/qos0_seq.png)

#### Message Flow - Qos1

![image](./_static/images/qos1_seq.png)

#### Message Flow - Qos2

![image](./_static/images/qos2_seq.png)

### Clean Session

The Client and Server can store Session state to enable reliable messaging to continue across a sequence of Network Connections.

The `Clean Session` flag can be assigned by CONNECT packet:

- If `Clean Session` is set to 0, the Server MUST resume communications with the Client based on state from the current Session (as identified by the Client identifier).
- If `Clean Session` is set to 1, the Client and server MUST discard any previous Session and start a new one.

### Keep Alive

The Keep Alive is a time interval measured in seconds. It can be assigned by CONNECT Packet.

It is the maximum time interval that is permitted to elapse between the point at which the Client finishes transmitting one Control Packet and the point it starts sending the next.

It is the responsibility of the Client to ensure that the interval between Control Packets being sent does not exceed the Keep Alive value. In the absence of sending any other Control Packets, the Client MUST send a PINGREQ Packet.

::: tip Tip
The allowed maximum keep alive timeout of EMQ X is 2 \* Keepalive
:::

### Will Message

The Will Message can be specified in CONNECT Packet. If the Connect request is accepted, a Will Message MUST be stored on the Server and associated with the Network Connection.

The Will Message MUST be published when the Network Connection is subsequently closed unless the Will Message has been deleted by the Server on receipt of a DISCONNECT Packet

### Retained Message

The Retain Flag is introduced by Publish Packet.

If the RETAIN flag is set to 1, in a PUBLISH Packet sent by a Client to a Server, the Server MUST store the Application Message and its QoS, so that it can be delivered to future subscribers whose subscriptions match its topic name.

E.g.: Publish a retained message to 'a/b/c':

    mosquitto_pub -r -q 1 -t a/b/c -m 'hello'

After the publish is successful, we are subscribed to the topic and can still receive the message:

    $ mosquitto_sub -t a/b/c -q 1
    hello

There are two way to clean Retained Message:

1. Send a empty message to retained topic:

       mosquitto_pub -r -q 1 -t a/b/c -m ''

2. Set the retained timeout in the configuration of broker

### MQTT Over WebSocket

In addition to supporting the TCP transport layer, the MQTT protocol also supports WebSocket as the transport layer.

Therefore, the browser can connect to the EMQ X message broker and communicate with other MQTT clients.

The Websocket connection must transport data with binary format, and carry the protocol field in the Header:

    Sec-WebSocket-Protocol: mqttv3.1 or mqttv3.1.1

### MQTT Client Libraries

#### EMQ X Team

EMQ X Team: [ https://github.com/emqx ](https://github.com/emqx)

| [ emqttc ](https://github.com/emqx/emqtt)        | Erlang MQTT Client which both support MQTTv3 and MQTTv5 |
| ------------------------------------------------ | ------------------------------------------------------- |
| [ CocoaMQTT ](https://github.com/emqx/CocoaMQTT) | MQTT for iOS and OS X written with Swift                |
| [ QMQTT ](https://github.com/emqx/qmqtt)         | MQTT Client for Qt                                      |

#### Eclipse Paho

Official Site: [ http://www.eclipse.org/paho/ ](http://www.eclipse.org/paho/)

Supported libraries: [ https://github.com/mqtt/mqtt.github.io/wiki/libraries ](https://github.com/mqtt/mqtt.github.io/wiki/libraries)

### MQTT vs. XMPP

The MQTT protocol is designed to be simple, lightweight, and flexible in routing. It will completely replace the XMPP protocol in the PC era in the field of mobile Internet IoT messaging:

1. The MQTT protocol only have one byte fixed header, two bytes keepalive message, which is lightweight and easy to encode/decode. The XMPP protocol is based on heavy XML, which has a large and excess description content for each message.
2. The MQTT protocol based only PUB/SUB communication mode. It is more flexible than XMPP's JID-based peer-to-peer message routing.
3. The MQTT protocol supports more flexible payload data format, such as JSON, String, Binary, etc.. But XMPP must encode the binary content with base64.
4. The MQTT protocol supports messaging acknowledgement and QoS mechanism, and the XMPP protocol does not define a similar mechanism. MQTT protocol has better message reliability than XMPP

## MQTT-SN Protocol

**MQTT-SN (MQTT for Sensor Network)** is designed to be as close as possible to MQTT, but is adapted to the peculiarities of a wireless com- munication environment such as low bandwidth, high link failures, short message length, etc. It is also optimized for the implementation on low-cost, battery-operated devices with limited processing and storage resources.

MQTT-SN Official Spec: [ http://mqtt.org/new/wp-content/uploads/2009/06/MQTT-SN_spec_v1.2.pdf ](http://mqtt.org/new/wp-content/uploads/2009/06/MQTT-SN_spec_v1.2.pdf)

### MQTT-SN vs. MQTT

Compared to MQTT, MQTT-SN is characterized by the following differences:

1. The CONNECT message is split into three messages. The two additional ones are optional and used to transfer the Will topic and the Will message to the server.
2. To cope with the short message length and the limited transmission bandwidth in wireless networks, the topic name in the PUBLISH messages is replaced by a short, two-byte long "topic id". A registration procedure is defined to allow clients to register their topic names with the server/gateway and obtain the corresponding topic ids. It is also used in the opposite direction to inform the client about the topic name and the corresponding topic id that will be included in a following PUBLISH message.
3. "Pre-defined" topic ids and "short" topic names are introduced, for which no registration is required. Pre-defined topic ids are also a two-byte long replacement of the topic name, their mapping to the topic names is however known in advance by both the client’s application and the gateway/server. Therefore both sides can start using pre-defined topic ids; there is no need for a registration as in the case of “normal” topic ids mentioned above.
4. A discovery procedure helps clients without a pre-configured server/gateway’s address to discover the actual network address of an operating server/gateway. Multiple gateways may be present at the same time within a single wireless network and can co-operate in a load-sharing or stand-by mode.
5. The semantic of a "clean session" is extended to the Will feature, i.e. not only client’s subscriptions are persistent, but also Will topic and Will message. A client can also modify its Will topic and Will message during a session.
6. A new offline keep-alive procedure is defined for the support of sleeping clients. With this procedure, battery-operated devices can go to a sleeping state during which all messages destined to them are buffered at the server/gateway and delivered later to them when they wake up.

### emqx-sn Plug-in

emqx-sn is a gateway plug-in of mqtt-sn protocol of EMQ X, which realizes the access function of mqtt-sn.

It is equivalent to an mqtt-sn gateway in the cloud, which is directly connected with EMQ X Broker.

#### Configuration

File: etc/plugins/emqx_sn.conf:

    mqtt.sn.port = 1884

    mqtt.sn.advertise_duration = 900

    mqtt.sn.gateway_id = 1

    mqtt.sn.username = mqtt_sn_user

    mqtt.sn.password = abc

| mqtt.sn.port               | The UDP port which emq-sn is listening on                             |
| -------------------------- | --------------------------------------------------------------------- |
| mqtt.sn.advertise_duration | The duration(seconds) that emq-sn broadcast ADVERTISE message through |
| mqtt.sn.gateway_id         |

> The MQTT-SN Gateway id in ADVERTISE message

mqtt.sn.username | Optional parameter. emqx-sn will connect EMQ core with this username. It is useful if any auth plug-in is enabled
mqtt.sn.password | Optional parameter. Pair with username above

#### Enable emqx-sn

    ./bin/emqx_ctl plugins load emqx_sn

### MQTT-SN Client Libraries

1. [ https://github.com/eclipse/paho.mqtt-sn.embedded-c/ ](https://github.com/eclipse/paho.mqtt-sn.embedded-c/)
2. [ https://github.com/ty4tw/MQTT-SN ](https://github.com/ty4tw/MQTT-SN)
3. [ https://github.com/njh/mqtt-sn-tools ](https://github.com/njh/mqtt-sn-tools)
4. [ https://github.com/arobenko/mqtt-sn ](https://github.com/arobenko/mqtt-sn)

## LWM2M Protocol

**LwM2M (Lightweight Machine-To-Machine)** is lightweight protcol for IoT that designed by OMA (Open Moblie Alliance). It provides device management and communication functions, especially for terminal devices with limited resources.

Official Documentations: [ http://www.openmobilealliance.org/wp/ ](http://www.openmobilealliance.org/wp/)

LwM2M is based on REST architecture and USES CoAP as the underlying transport protocol.

It is carried on UDP or SMS, so the message structure is simple and small, and it is also applicable in the environment where network resources are limited and the device cannot always be online.

![image](./_static/images/lwm2m_protocols.png)

There are two primary roles for LwM2M, that is `LwM2M Server` and `LwM2M Client` .

As the server side, the LwM2M Server is depolyed at the Services Provider or Network Provider. It defines two types of servers:

- LwM2M Bootstrap Server: Not yet implemented in EMQ X
- LwM2M Server: The connection acceptor for LwM2M protcol. emqx-lwm2m has implement it over UDP transport layer only, not yet SMS

As the client side, the LwM2M Client is presented on terminal devices.

The LwM2M has defined 4 types Interfaces for communication each others:

1. **Bootstrap:** The Bootstrap Interface is used to provision essential information into the LwM2M Client to enable the LwM2M Client to perform the "Register" operation with one or more LwM2M Servers
2. **Client Registration:** The Client Registration Interface is used by a LwM2M Client to register with one or more LwM2M Servers, maintain each registration, and de-register from a LwM2M Server. When registering, the LwM2M Client performs the "Register" operation and provides the properties required by the LwM2M Server
3. **Device Management and Service Enablement:** It is used by the LwM2M Server to access Object Instances and Resources available from a registered LwM2M Client. The interface provides this access through the use of "Create", "Read", "Read-Composite", "Write", "Write-Composite", "Delete", "Execute", "Write-Attributes", or "Discover" operations.
4. **Information Reporting:** It is used by a LwM2M Server to observe any changes in a Resource on a registered LwM2M Client, receiving notifications when new values are available.

![image](./_static/images/lwm2m_arch.png)

LwM2M abstracts the services on devices into objects and resources, and defines the properties and functions of various objects in XML files.

See: [ http://www.openmobilealliance.org/wp/OMNA/LwM2M/LwM2MRegistry.html ](http://www.openmobilealliance.org/wp/OMNA/LwM2M/LwM2MRegistry.html)

LwM2M pre-defined 8 types object to implement business requirements:

- Security
- Server
- Access Control
- Device
- Connectivity Monitoring
- Firmware
- Location
- Connectivity Statistics

### emqx-lwm2m Plug-in

[ emqx-lwm2m ](https://github.com/emqx/emqx-lwm2m) is a LwM2M Server implement for EMQ X. It support a LwM2M Client connecting to EMQ X system, reporting data, etc.

### Adapt to EMQ X

The emqx-lwm2m can deliver a message to LwM2M Client with specified Topic, it configured at `etc/plugins/emqx_lwm2m.conf` :

    # The topic subscribed by the lwm2m client after it is connected
    # Placeholders supported:
    #    '%e': Endpoint Name
    #    '%a': IP Address
    lwm2m.topics.command = lwm2m/%e/command

The response topic specifed with:

    # The topic to which the lwm2m client's response is published
    lwm2m.topics.response = lwm2m/%e/resp

It is note that the payload of MQTT is string with JSON format. The details can be found README.md at [ emqx-lwm2m ](https://github.com/emqx/emqx-lwm2m)

#### Configurations

File: etc/plugins/emqx_lwm2m.conf:

    lwm2m.port = 5683

    lwm2m.certfile = etc/certs/cert.pem

    lwm2m.keyfile = etc/certs/key.pem

    lwm2m.xml_dir =  etc/lwm2m_xml

| lwm2m.port     | UDP port of the LwM2M Gateway                      |
| -------------- | -------------------------------------------------- |
| lwm2m.certfile | Certificate file for DTLS                          |
| lwm2m.keyfile  | Key file for DTLS                                  |
| lwm2m.xml_dir  | Dir where the object definition files can be found |

#### Enable emqx-lwm2m

    ./bin/emqx_ctl plugins load emqx_lwm2m

### LwM2M Libraries

- [ https://github.com/eclipse/wakaama ](https://github.com/eclipse/wakaama)
- [ https://github.com/OpenMobileAlliance/OMA-LWM2M-DevKit ](https://github.com/OpenMobileAlliance/OMA-LWM2M-DevKit)
- [ https://github.com/AVSystem/Anjay ](https://github.com/AVSystem/Anjay)
- [ http://www.eclipse.org/leshan/ ](http://www.eclipse.org/leshan/)

## TCP Protocol

The TCP Protocol is a transparent transmission protocol, that is designed by EMQ X Team to accept some private protcol devices access and communication with other types device

::: tip Tip
Supported above v3.2.2
:::

### Packet Structure

The Control Packes consist of two parts: **Fixed Header** and **Payload** .

There are two bytes in front of the payload that mark payload length:

      +----------------+--------------+
      | Fixed Header   |    1 Byte    |
      +----------------+--------------+
      | Len of Payloa  |    2 Bytes   |
      +----------------+--------------+
      | Payload        |    N Bytes   |
      +----------------+--------------+


::: tip Tip
Some packet types have no payload fields, so the entire package has only one byte
:::

The first 4 Bits in the Fixed Header represent **Frame Type** , which is currently supported:

| Name       | Value | Direction of Flow   | Description                                                                  |
| ---------- | ----- | ------------------- | ---------------------------------------------------------------------------- |
| CONNECT    | 1     | Client --> Server   | Connect to Server. It must be a first packet that the client sends to server |
| CONNACK    | 2     | Server --> Client   | Connect acknowledgement                                                      |
| DATATRANS  | 3     | Client \<==> Server | A data communication packet between client and server                        |
| PING       | 4     | Client --> Server   | Ping request                                                                 |
| PONG       | 5     | Server --> Client   | Ping response                                                                |
| DISCONNECT | 6     | Client --> Server   | Client to send a DISCONNECT packet to end current connection with server     |
| Reserved   | 7-15  | Reserved            | Reserved                                                                     |

#### CONNECT

Connect Packet. The Frame Type is 2#0001.

The first 4 bits of Flags represents the **Protocol Version** is currently 1, namely 2#0001. So CONNECT packet Fixed Header is **0x11**

The Payload contains all requirement fields to create a connection. It must be given in the following order; otherwise, it is an illegal packet and immediately disconnect the TCP link:

    Len       Keepalive[x] ClientId[x]  Username  Password
    UINT(2)   UINT(1)      STR(n)       STR(n)    STR(n)

Keepalive and ClientId is required；Username and Password is optional.

So，two examples packet content can be given with followings:

    # Keepalive = 60
    # ClientId  = abcd
    # Username, Password are undefined
    0x10 00 07 3c 00 04 61 62 63 64

    # Keepalive = 60
    # ClientId  = abcd
    # Username  = abcd
    # Password  = abcd
    0x10 00 13 3c 00 04 61 62 63 64 00 04 61 62 63 64 00 04 61 62 63 64

#### CONNACK

Connect acknowledgement. The Frame Type is 2#0010.

The first 4 bits of Flags represents the **ACK Code** :

| Name       | Value | Description                  |
| ---------- | ----- | ---------------------------- |
| SUCCESSFUL | 0     | Connect Successfully         |
| AUTHFAILED | 1     | Authentication failure       |
| ILLEGALVER | 2     | Unsupported protocol version |
| Reserved   | 3-15  | Reserved                     |

The Payload is a acknowledgement **Message** string.

    Message
     STR(n)

So, there are two examples to indicate the CONNACK conent details:

    # ACK Code = 1
    # Message = 'Connect Sucessfully'
    0x20 00 14 43 6f 6e 6e 65 63 74 20 53 75 63 63 65 73 73 66 75 6c 6c 79

    # ACK Code = 1
    # Message = ''
    0x21 00 00

#### DATATRANS

Data Transmission Packet. The Frame Type is 2#0011.

The first 2 bits of Flags is **Qos(Quality of Services)** . It is a constant value 0 now. The last two bits are reserved bits. So the Fixed Header of DATATRANS is 0x30

The Payload conent is a data for transparent transmission

    Len     Payload
    UINT(2) BIN

::: tip Tip
The maximum size of payload data is 65535
:::

So, If it send a 'abcd' string, the DATATRANS is:

    0x30 00 04 61 62 63 63

#### PING

Ping request packet. The Frame Type is 2#0100. Flags is 0. So the Fixed Header is 0x40

Payload is absence

A example for PING packet:

    0x40

#### PONG

PING response packet. The Frame Type is 2#0101. Flags is 0. So the Fixed Header is 0x50

Payload is absence

A example for PONG packet:

    0x50

#### DISCONNECT

Disconnect packet. The Frame Type is 2#0111. Flags is 0. So the Fixed Header is 0x60

Payload is absence

A example for DISCONNECT packet:

    0x60

### emqx-tcp Plug-in

The TCP transparent transmission protocol implemented by **emqx_tcp**

#### Adapt to EMQ X

The **emqx-tcp** plug-in can publish the uplink messages of device with `up_topic` . This option defined at `etc/plugins/emqx_tcp.conf` :

    tcp.proto.up_topic = tcp/%c/up

Similarly, The `dn_topic` option can be configured to receive downlink messages:

    tcp.proto.dn_topic = tcp/%c/dn

#### Listeners

The emqx-tcp can configure a list of listeners for accepting TCP/SSL devices in the `etc/plugins/emqx_tcp.conf` :

    tcp.listener.external = 0.0.0.0:8090

    tcp.listener.external.acceptors = 8

    tcp.listener.external.max_connections = 1024000

    ## SSL
    tcp.listener.ssl.external = 0.0.0.0:8091

    tcp.listener.ssl.external.acceptors = 8

    tcp.listener.ssl.external.max_connections = 1024000
