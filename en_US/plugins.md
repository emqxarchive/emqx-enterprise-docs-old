# Plugins

Through module registration and hooks (Hooks) mechanism user can develop plugin to customize authentication and service functions for EMQ X.

The official plug-ins provided by EMQ X include:

| Plugin                                                                    | Configuration file                    | Description                        |
| ------------------------------------------------------------------------- | ------------------------------------- | ---------------------------------- |
| [ emqx_dashboard ](https://github.com/emqx/emqx-dashboard) \+             | etc/plugins/emqx_dashbord.conf        | Web dashboard Plugin (Default)     |
| [ emqx_management ](https://github.com/emqx/emqx-management) \+           | etc/plugins/emqx_management.conf      | HTTP API and CLI Management Plugin |
| [ emqx_psk_file ](https://github.com/emqx/emqx-psk-file) \+               | etc/plugins/emqx_psk_file.conf        | PSK support                        |
| [ emqx_web_hook ](https://github.com/emqx/emqx-web-hook) \+               | etc/plugins/emqx_web_hook.conf        | Web Hook Plugin                    |
| [ emqx_lua_hook ](https://github.com/emqx/emqx-lua-hook) \+               | etc/plugins/emqx_lua_hook.conf        | Lua Hook Plugin                    |
| [ emqx_retainer ](https://github.com/emqx/emqx-retainer) \+               | etc/plugins/emqx_retainer.conf        | Retain Message storage module      |
| [ emqx_rule_engine ](https://github.com/emqx/emqx-rule-engine) \+         | etc/plugins/emqx_rule_engine.conf     | Rule engine                        |
| [ emqx_bridge_mqtt ](https://github.com/emqx/emqx-bridge-mqtt) \+         | etc/plugins/emqx_bridge_mqtt.conf     | MQTT Message Bridge Plugin         |
| [ emqx_delayed_publish ](https://github.com/emqx/emqx-delayed-publish) \+ | etc/plugins/emqx_delayed_publish.conf | Delayed publish support            |
| [ emqx_coap ](https://github.com/emqx/emqx-coap) \+                       | etc/plugins/emqx_coap.conf            | CoAP protocol support              |
| [ emqx_lwm2m ](https://github.com/emqx/emqx-lwm2m) \+                     | etc/plugins/emqx_lwm2m.conf           | LwM2M protocol support             |
| [ emqx_sn ](https://github.com/emqx/emqx-sn) \+                           | etc/plugins/emqx_sn.conf              | MQTT-SN protocol support           |
| [ emqx_stomp ](https://github.com/emqx/emqx-stomp) \+                     | etc/plugins/emqx_stomp.conf           | Stomp protocol support             |
| [ emqx_recon ](https://github.com/emqx/emqx-recon) \+                     | etc/plugins/emqx_recon.conf           | Recon performance debugging        |
| [ emqx_reloader ](https://github.com/emqx/emqx-reloader) \+               | etc/plugins/emqx_reloader.conf        | Hot load plugin                    |
| [ emqx_plugin_template ](https://github.com/emqx/emqx-plugin-template) \+ | etc/plugins/emqx_plugin_template.conf | plugin develop template            |

There are four ways to load plugins:

1. Default loading
2. Start and stop plugin on command line
3. Start and stop plugin on Dashboard
4. Start and stop plugin by calling management API

**Default loading**

If a plugin needs to start with the broker, add this plugin in `data/loaded_plugins` . For example, the plugins that are loaded by default are:

    emqx_management.
    emqx_rule_engine.
    emqx_recon.
    emqx_retainer.
    emqx_dashboard.

**Start and stop plugin on command line**

When the EMQ X is running, plugin list can be displayed and plugins can be loaded/unloaded using CLI command:

    ## Display a list of all available plugins
    ./bin/emqx_ctl plugins list

    ## Load a plugin
    ./bin/emqx_ctl plugins load emqx_auth_username

    ## Unload a plugin
    ./bin/emqx_ctl plugins unload emqx_auth_username

    ## Reload a plugin
    ./bin/emqx_ctl plugins reload emqx_auth_username

**Start and stop plugin on Dashboard**

If Dashboard plugin is started (by default), the plugins can be also managed on the dashboard. the managing page can be found under `http://localhost:18083/plugins` .

## Dashboard Plugin

[ emqx_dashboard ](https://github.com/emqx/emqx-dashboard) is the web management console for the EMQ X broker, which is enabled by default. When EMQ X starts successfully, user can access it by visiting `http://localhost:18083` with the default username/password: admin/public.

> The basic information, statistics, and load status of the EMQ X broker, as well as the current client list (Connections), Sessions, Routing Table (Topics), and Subscriptions can be queried through dashboard.

![image](./_static/images/dashboard.png)

In addition, dashboard provides a set of REST APIs for front-end calls. See `Dashboard -> HTTP API` for details.

### Dashboard plugin settings

etc/plugins/emqx_dashboard.conf:

    ## Dashboard default username/password
    dashboard.default_user.login = admin
    dashboard.default_user.password = public

    ## Dashboard HTTP service Port Configuration
    dashboard.listener.http = 18083
    dashboard.listener.http.acceptors = 2
    dashboard.listener.http.max_clients = 512

    ## Dashboard HTTPS service Port Configuration
    ## dashboard.listener.https = 18084
    ## dashboard.listener.https.acceptors = 2
    ## dashboard.listener.https.max_clients = 512
    ## dashboard.listener.https.handshake_timeout = 15s
    ## dashboard.listener.https.certfile = etc/certs/cert.pem
    ## dashboard.listener.https.keyfile = etc/certs/key.pem
    ## dashboard.listener.https.cacertfile = etc/certs/cacert.pem
    ## dashboard.listener.https.verify = verify_peer
    ## dashboard.listener.https.fail_if_no_peer_cert = true

## HTTP API and CLI Management Plugin

[ emqx_management ](https://github.com/emqx/emqx-management) is the HTTP API and CLI management plugin of the EMQ X broker，which is enabled by default. When EMQ X starts successfully, users can query the current client list and other operations via the HTTP API and CLI provided by this plugin. For details see `rest_api` and `commands` .

### HTTP API and CLI Management Configuration

etc/plugins/emqx_management.conf:

    ## Max Row Limit
    management.max_row_limit = 10000

    ## Default Application Secret
    # management.application.default_secret = public

    ## Management HTTP Service Port Configuration
    management.listener.http = 8080
    management.listener.http.acceptors = 2
    management.listener.http.max_clients = 512
    management.listener.http.backlog = 512
    management.listener.http.send_timeout = 15s
    management.listener.http.send_timeout_close = on

    ## Management HTTPS Service Port Configuration
    ## management.listener.https = 8081
    ## management.listener.https.acceptors = 2
    ## management.listener.https.max_clients = 512
    ## management.listener.https.backlog = 512
    ## management.listener.https.send_timeout = 15s
    ## management.listener.https.send_timeout_close = on
    ## management.listener.https.certfile = etc/certs/cert.pem
    ## management.listener.https.keyfile = etc/certs/key.pem
    ## management.listener.https.cacertfile = etc/certs/cacert.pem
    ## management.listener.https.verify = verify_peer
    ## management.listener.https.fail_if_no_peer_cert = true

## PSK Authentication Plugin

[ emqx_psk_file ](https://github.com/emqx/emqx-psk-file) mainly provides PSK support that aimes to implement connection authentication through PSK when the client establishes a TLS/DTLS connection.

### PSK Authentication Plugin Configuration

etc/plugins/emqx_psk_file.conf:

    psk.file.path = etc/psk.txt

## WebHook Plugin

[ emqx_web_hook ](https://github.com/emqx/emqx-web-hook) can send all EMQ X events and messages to the specified HTTP server.

### WebHook plugin configuration

etc/plugins/emqx_web_hook.conf:

    ## Callback Web Server Address
    web.hook.api.url = http://127.0.0.1:8080

    ## Encode message payload field
    ## Value: undefined | base64 | base62
    ## Default: undefined (Do not encode)
    ## web.hook.encode_payload = base64

    ## Message and event configuration
    web.hook.rule.client.connected.1     = {"action": "on_client_connected"}
    web.hook.rule.client.disconnected.1  = {"action": "on_client_disconnected"}
    web.hook.rule.client.subscribe.1     = {"action": "on_client_subscribe"}
    web.hook.rule.client.unsubscribe.1   = {"action": "on_client_unsubscribe"}
    web.hook.rule.session.created.1      = {"action": "on_session_created"}
    web.hook.rule.session.subscribed.1   = {"action": "on_session_subscribed"}
    web.hook.rule.session.unsubscribed.1 = {"action": "on_session_unsubscribed"}
    web.hook.rule.session.terminated.1   = {"action": "on_session_terminated"}
    web.hook.rule.message.publish.1      = {"action": "on_message_publish"}
    web.hook.rule.message.deliver.1      = {"action": "on_message_deliver"}
    web.hook.rule.message.acked.1        = {"action": "on_message_acked"}

## Lua Plugin

[ emqx_lua_hook ](https://github.com/emqx/emqx-lua-hook) sends all events and messages to the specified Lua function. See its README for specific use.

## Retainer Plugin

[ emqx_retainer ](https://github.com/emqx/emqx-retainer) is set to start by default and provides Retained type message support for EMQ X. It stores the Retained messages for all topics in the cluster's database and posts the message when the client subscribes to the topic

### Retainer Plugin Configuration

etc/plugins/emqx_retainer.conf:

    ## retained Message storage method
    ##  - ram: memory only
    ##  - disc: memory and disk
    ##  - disc_only: disk only
    retainer.storage_type = ram

    ## Maximum number of storage (0 means unrestricted)
    retainer.max_retained_messages = 0

    ## Maximum storage size for single message
    retainer.max_payload_size = 1MB

    ## Expiration time, 0 means never expired
    ## Unit:  h hour; m minute; s second.For example, 60m means 60 minutes.
    retainer.expiry_interval = 0

## MQTT Message Bridge Plugin

The concept of **Bridge** is that EMQ X forwards messages of some of its topics to another MQTT Broker in some way.

Difference between **Bridge** and **cluster** is that bridge does not replicate topic trees and routing tables, a bridge only forwards MQTT messages based on Bridge rules.

Currently the Bridge methods supported by EMQ X are as follows:

- RPC bridge: RPC Bridge only supports message forwarding and does not support subscribing to the topic of remote nodes to synchronize data.
- MQTT Bridge: MQTT Bridge supports both forwarding and data synchronization through subscription topic

In EMQ X, bridge is configured by modifying `etc/plugins/emqx_bridge_mqtt.conf` . EMQ X distinguishes between different bridges based on different names. E.g:

    ## Bridge address: node name for local bridge, host:port for remote.
    bridge.mqtt.aws.address = 127.0.0.1:1883

This configuration declares a bridge named `aws` and specifies that it is bridged to the MQTT server of `127.0.0.1:1883` by MQTT mode.

In case of creating multiple bridges, it is convenient to replicate all configuration items of the first bridge, and modify the bridge name and other configuration items if necessary (such as bridge.mqtt.$name.address, where $name refers to the name of bridge)

### MQTT Bridge Plugin Configuration

etc/plugins/emqx_bridge_mqtt.conf

    ## Bridge Address: Use node name (nodename@host) for rpc Bridge, and host:port for mqtt connection
    bridge.mqtt.aws.address = emqx2@192.168.1.2

    ## Forwarding topics of the message
    bridge.mqtt.aws.forwards = sensor1/#,sensor2/#

    ## bridged mountpoint
    bridge.mqtt.aws.mountpoint = bridge/emqx2/${node}/

    ## Bridge Address: Use node name for rpc Bridge, use host:port for mqtt connection
    bridge.mqtt.aws.address = 192.168.1.2:1883

    ## Bridged Protocol Version
    ## Enumeration value: mqttv3 | mqttv4 | mqttv5
    bridge.mqtt.aws.proto_ver = mqttv4

    ## mqtt client's client_id
    bridge.mqtt.aws.client_id = bridge_emq

    ## mqtt client's clean_start field
    ## Note: Some MQTT Brokers need to set the clean_start value as `true`
    bridge.mqtt.aws.clean_start = true

    ##  mqtt client's username field
    bridge.mqtt.aws.username = user

    ## mqtt client's password field
    bridge.mqtt.aws.password = passwd

    ## Whether the mqtt client uses ssl to connect to a remote serve or not
    bridge.mqtt.aws.ssl = off

    ## CA Certificate of Client SSL Connection (PEM format)
    bridge.mqtt.aws.cacertfile = etc/certs/cacert.pem

    ## SSL certificate of Client SSL connection
    bridge.mqtt.aws.certfile = etc/certs/client-cert.pem

    ## Key file of Client SSL connection
    bridge.mqtt.aws.keyfile = etc/certs/client-key.pem

    ## SSL encryption
    bridge.mqtt.aws.ciphers = ECDHE-ECDSA-AES256-GCM-SHA384,ECDHE-RSA-AES256-GCM-SHA384

    ## TTLS PSK password
    ## Note 'listener.ssl.external.ciphers' and 'listener.ssl.external.psk_ciphers' cannot be configured at the same time
    ##
    ## See 'https://tools.ietf.org/html/rfc4279#section-2'.
    ## bridge.mqtt.aws.psk_ciphers = PSK-AES128-CBC-SHA,PSK-AES256-CBC-SHA,PSK-3DES-EDE-CBC-SHA,PSK-RC4-SHA

    ## Client's heartbeat interval
    bridge.mqtt.aws.keepalive = 60s

    ## Supported TLS version
    bridge.mqtt.aws.tls_versions = tlsv1.2,tlsv1.1,tlsv1

    ## Forwarding topics of the message
    bridge.mqtt.aws.forwards = sensor1/#,sensor2/#

    ## Bridged mountpoint
    bridge.mqtt.aws.mountpoint = bridge/emqx2/${node}/

    ## Subscription topic for Bridge
    bridge.mqtt.aws.subscription.1.topic = cmd/topic1

    ## Subscription qos for Bridge
    bridge.mqtt.aws.subscription.1.qos = 1

    ## Subscription topic for Bridge
    bridge.mqtt.aws.subscription.2.topic = cmd/topic2

    ## Subscription qos for Bridge
    bridge.mqtt.aws.subscription.2.qos = 1

    ## Bridge reconnection interval
    ## Default: 30s
    bridge.mqtt.aws.reconnect_interval = 30s

    ## QoS1 message retransmission interval
    bridge.mqtt.aws.retry_interval = 20s

    ## Inflight Size.
    bridge.mqtt.aws.max_inflight_batches = 32

    ## emqx_bridge internal number of messages used for batch
    bridge.mqtt.aws.queue.batch_count_limit = 32

    ##  emqx_bridge internal number of message bytes used for batch
    bridge.mqtt.aws.queue.batch_bytes_limit = 1000MB

    ## The path for placing replayq queue. If the item is not specified in the configuration, then replayq will run in `mem-only` mode and messages will not be cached on disk.
    bridge.mqtt.aws.queue.replayq_dir = data/emqx_emqx2_bridge/

    ## Replayq data segment size
    bridge.mqtt.aws.queue.replayq_seg_bytes = 10MB

## Delayed Publish Plugin

[ emqx_delayed_publish ](https://github.com/emqx/emqx-delayed-publish) provides the function to delay publishing messages. When the client posts a message to EMQ X using the special topic prefix `$delayed/\<seconds>/` , EMQ X will publish this message after \<seconds> seconds.

## CoAP Protocol Plugin

[ emqx_coap ](https://github.com/emqx/emqx-coap) provides support for the CoAP protocol (RFC 7252)。

### CoAP protocol Plugin Configuration

etc/plugins/emqx_coap.conf:

    coap.port = 5683

    coap.keepalive = 120s

    coap.enable_stats = off

DTLS can be enabled if the following two configuration items are set:

    ## Listen port for DTLS
    coap.dtls.port = 5684

    coap.dtls.keyfile = {{ platform_etc_dir }}/certs/key.pem
    coap.dtls.certfile = {{ platform_etc_dir }}/certs/cert.pem

    ## DTLS options
    ## coap.dtls.verify = verify_peer
    ## coap.dtls.cacertfile = {{ platform_etc_dir }}/certs/cacert.pem
    ## coap.dtls.fail_if_no_peer_cert = false

### Test the CoAP Plugin

A CoAP client is necessary to test CoAP plugin. In following example the [ libcoap ](https://github.com/obgm/libcoap) is used.

    yum install libcoap

    % coap client publish message
    coap-client -m put -e "qos=0&retain=0&message=payload&topic=hello" coap://localhost/mqtt

## LwM2M Protocol Plugin

[ emqx_lwm2m ](https://github.com/emqx/emqx-lwm2m) provides support for the LwM2M protocol.

### LwM2M plugin configuration

etc/plugins/emqx_lwm2m.conf:

    ## LwM2M listening port
    lwm2m.port = 5683

    ## Lifetime Limit
    lwm2m.lifetime_min = 1s
    lwm2m.lifetime_max = 86400s

    ## `time window` length under Q Mode Mode, in seconds.
    ## Messages that exceed the window will be cached
    #lwm2m.qmode_time_window = 22

    ## Whether LwM2M is deployed after coaproxy
    #lwm2m.lb = coaproxy

    ## Actively observe all objects after the device goes online
    #lwm2m.auto_observe = off

    ## the subscribed topic from EMQ X after client register succeeded
    ## Placeholder:
    ##    '%e': Endpoint Name
    ##    '%a': IP Address
    lwm2m.topics.command = lwm2m/%e/dn/#

    ## client response message to EMQ X topic
    lwm2m.topics.response = lwm2m/%e/up/resp

    ## client notify message to EMQ X topic
    lwm2m.topics.notify = lwm2m/%e/up/notify

    ## client register message to EMQ X topic
    lwm2m.topics.register = lwm2m/%e/up/resp

    # client update message to EMQ X topic
    lwm2m.topics.update = lwm2m/%e/up/resp

    # xml file location defined by object
    lwm2m.xml_dir =  etc/lwm2m_xml

DTLS support can be enabled with the following configuration:

    # DTLS Certificate Configuration
    lwm2m.certfile = etc/certs/cert.pem
    lwm2m.keyfile = etc/certs/key.pem

## MQTT-SN Protocol Plugin

[ emqx_sn ](https://github.com/emqx/emqx-sn) provides support for the MQTT-SN protocol

### MQTT-SN protocol plugin configuration

etc/plugins/emqx_sn.conf:

    mqtt.sn.port = 1884

## Stomp Protocol Plugin

[ emqx_stomp ](https://github.com/emqx/emqx-stomp) provides support for the Stomp protocol. Clients connect to EMQ X through Stomp 1.0/1.1/1.2 protocol, publish and subscribe to MQTT message.

### Stomp plugin configuration

::: tip Tip
Stomp protocol port: 61613
:::

etc/plugins/emqx_stomp.conf:

    stomp.default_user.login = guest

    stomp.default_user.passcode = guest

    stomp.allow_anonymous = true

    stomp.frame.max_headers = 10

    stomp.frame.max_header_length = 1024

    stomp.frame.max_body_length = 8192

    stomp.listener = 61613

    stomp.listener.acceptors = 4

    stomp.listener.max_clients = 512

## Recon Performance Debugging Plugin

[ emqx_recon ](https://github.com/emqx/emqx-recon) integrates the recon performance tuning library to view status information about the current system, for example:

    ./bin/emqx_ctl recon

    recon memory                 #recon_alloc:memory/2
    recon allocated              #recon_alloc:memory(allocated_types, current|max)
    recon bin_leak               #recon:bin_leak(100)
    recon node_stats             #recon:node_stats(10, 1000)
    recon remote_load Mod        #recon:remote_load(Mod)

### Recon Plugin Configuration

etc/plugins/emqx_recon.conf:

    %% Garbage Collection: 10 minutes
    recon.gc_interval = 600

## Reloader Hot Reload Plugin

[ emqx_reloader ](https://github.com/emqx/emqx-reloader) is used for code hot-upgrade during impelementation and debugging. After loading this plug-in, EMQ X updates the codes automatically according to the configuration interval.

A CLI command is also provided to force a module to reload:

    ./bin/emqx_ctl reload \<Module>

::: tip Tip
This plugin is not recommended for production environments.
:::

### Reloader Plugin Configuration

etc/plugins/emqx_reloader.conf:

    reloader.interval = 60

    reloader.logfile = log/reloader.log

## Plugin Development Template

[ emqx_plugin_template ](https://github.com/emqx/emqx-plugin-template) is an EMQ X plugin template and provides no functionality by itself.

When developers need to customize a plugin, they can view this plugin's code and structure to deliver a standard EMQ X plugin faster. The plugin is actually a normal `Erlang Application` with the configuration file: `etc/${PluginName}.config` .

## EMQ X R3.2 Plugin Development

### Create a Plugin Project

For creating a new plugin project please refer to the [ emqx_plugin_template ](https://github.com/emqx/emqx-plugin-template) . .. 

::: tip Tip
The tag `-emqx_plugin(?MODULE).` must be added to the `\<plugin name>_app.erl` file to indicate that this is a plugin for EMQ X.
:::

### Create an Authentication/Access Control Module

A demo of authentication module - emqx_auth_demo.erl

    -module(emqx_auth_demo).

    -export([ init/1
            , check/2
            , description/0
            ]).

    init(Opts) -> {ok, Opts}.

    check(_Credentials = #{client_id := ClientId, username := Username, password := Password}, _State) ->
        io:format("Auth Demo: clientId=~p, username=~p, password=~p~n", [ClientId, Username, Password]),
        ok.

    description() -> "Auth Demo Module".

A demo of access control module - emqx_acl_demo.erl

    -module(emqx_acl_demo).

    -include_lib("emqx/include/emqx.hrl").

    %% ACL callbacks
    -export([ init/1
            , check_acl/5
            , reload_acl/1
            , description/0
            ]).

    init(Opts) ->
        {ok, Opts}.

    check_acl({Credentials, PubSub, _NoMatchAction, Topic}, _State) ->
        io:format("ACL Demo: ~p ~p ~p~n", [Credentials, PubSub, Topic]),
        allow.

    reload_acl(_State) ->
        ok.

    description() -> "ACL Demo Module".

Registration of authentication, access control module - emqx_plugin_template_app.erl

    ok = emqx:hook('client.authenticate', fun emqx_auth_demo:check/2, []),
    ok = emqx:hook('client.check_acl', fun emqx_acl_demo:check_acl/5, []).

### Hooks

Events of client's online and offline, topic subscription, message sending and receiving can be handled through hooks.

emqx_plugin_template.erl:

    %% Called when the plugin application start
    load(Env) ->
        emqx:hook('client.authenticate', fun ?MODULE:on_client_authenticate/2, [Env]),
        emqx:hook('client.check_acl', fun ?MODULE:on_client_check_acl/5, [Env]),
        emqx:hook('client.connected', fun ?MODULE:on_client_connected/4, [Env]),
        emqx:hook('client.disconnected', fun ?MODULE:on_client_disconnected/3, [Env]),
        emqx:hook('client.subscribe', fun ?MODULE:on_client_subscribe/3, [Env]),
        emqx:hook('client.unsubscribe', fun ?MODULE:on_client_unsubscribe/3, [Env]),
        emqx:hook('session.created', fun ?MODULE:on_session_created/3, [Env]),
        emqx:hook('session.resumed', fun ?MODULE:on_session_resumed/3, [Env]),
        emqx:hook('session.subscribed', fun ?MODULE:on_session_subscribed/4, [Env]),
        emqx:hook('session.unsubscribed', fun ?MODULE:on_session_unsubscribed/4, [Env]),
        emqx:hook('session.terminated', fun ?MODULE:on_session_terminated/3, [Env]),
        emqx:hook('message.publish', fun ?MODULE:on_message_publish/2, [Env]),
        emqx:hook('message.deliver', fun ?MODULE:on_message_deliver/3, [Env]),
        emqx:hook('message.acked', fun ?MODULE:on_message_acked/3, [Env]),
        emqx:hook('message.dropped', fun ?MODULE:on_message_dropped/3, [Env]).

Available hooks description:

| Hooks                | Description                      |
| -------------------- | -------------------------------- |
| client.authenticate  | connection authentication        |
| client.check_acl     | ACL validation                   |
| client.connected     | client online                    |
| client.disconnected  | client disconnected              |
| client.subscribe     | subscribe topic by client        |
| client.unsubscribe   | unsubscribe topic by client      |
| session.created      | session created                  |
| session.resumed      | session resumed                  |
| session.subscribed   | session after topic subscribed   |
| session.unsubscribed | session after topic unsubscribed |
| session.terminated   | session terminated               |
| message.publish      | MQTT message publish             |
| message.deliver      | MQTT message deliver             |
| message.acked        | MQTT message acknowledged        |
| message.dropped      | MQTT message dropped             |

### Register CLI Command

Demo module for extending command line - emqx_cli_demo.erl

    -module(emqx_cli_demo).

    -export([cmd/1]).

    cmd(["arg1", "arg2"]) ->
        emqx_cli:print("ok");

    cmd(_) ->
        emqx_cli:usage([{"cmd arg1 arg2", "cmd demo"}]).

Register command line module - emqx_plugin_template_app.erl

    ok = emqx_ctl:register_command(cmd, {emqx_cli_demo, cmd}, []),

After the plugin is loaded，a new CLI command is added to `./bin/emqx_ctl` ：

    ./bin/emqx_ctl cmd arg1 arg2

### Plugin Configuration File

The plugin comes with a configuration file placed in `etc/${plugin_name}.conf|config` . EMQ X supports two plugin configuration formats:

1.  Erlang native configuration file format - `${plugin_name}.config` :

        [

    {plugin_name, [
    {key, value}
    ]}
    ].

2.  sysctl's `k = v` universal forma - `${plugin_name}.conf` :

        plugin_name.key = value

::: tip Tip
`k = v` format configuration requires the plugin developer to create a `priv/plugin_name.schema` mapping file.
:::

### Compile and Release Plugin

1. clone emqx-rel project:


        git clone https://github.com/emqx/emqx-rel.git

2. Add dependency in rebar.config:


        {deps,
           [ {plugin_name, {git, "url_of_plugin", {tag, "tag_of_plugin"}}}
           , ....
           ....
           ]
        }

3. The relx paragraph in rebar.config is added:


        {relx,
            [...
            , ...
            , {release, {emqx, git_describe},
               [
                 {plugin_name, load},
               ]
              }
            ]
        }
