# Tuning Guide

EMQ X R2 scaled to 1.3 million concurrent MQTT connections on a 8 Core/32GB CentOS server.

Tuning the Linux Kernel, Networking, Erlang VM and the EMQ broker for one million concurrent MQTT connections.

## Linux Kernel Tuning

The system-wide limit on max opened file handles:

    # 2 millions system-wide
    sysctl -w fs.file-max=2097152
    sysctl -w fs.nr_open=2097152
    echo 2097152 > /proc/sys/fs/nr_open

The limit on opened file handles for current session:

    ulimit -n 1048576

### /etc/sysctl.conf

Add the ‘fs.file-max’ to /etc/sysctl.conf, make the changes permanent:

    fs.file-max = 1048576

### /etc/security/limits.conf

Persist the maximum number of opened file handles for users in /etc/security/limits.conf:

    emqx      soft   nofile      1048576
    emqx      hard   nofile      1048576

::: tip Tip
Under ubuntu, '/etc/systemd/system.conf' has to to be modified:
:::

    DefaultLimitNOFILE=1048576

## TCP Network Tuning

Increase number of incoming connections backlog:

    sysctl -w net.core.somaxconn=32768
    sysctl -w net.ipv4.tcp_max_syn_backlog=16384
    sysctl -w net.core.netdev_max_backlog=16384

Local port range :

    sysctl -w net.ipv4.ip_local_port_range='1000 65535'

TCP Socket read/write buffer:

    sysctl -w net.core.rmem_default=262144
    sysctl -w net.core.wmem_default=262144
    sysctl -w net.core.rmem_max=16777216
    sysctl -w net.core.wmem_max=16777216
    sysctl -w net.core.optmem_max=16777216

    #sysctl -w net.ipv4.tcp_mem='16777216 16777216 16777216'
    sysctl -w net.ipv4.tcp_rmem='1024 4096 16777216'
    sysctl -w net.ipv4.tcp_wmem='1024 4096 16777216'

TCP connection tracking:

    sysctl -w net.nf_conntrack_max=1000000
    sysctl -w net.netfilter.nf_conntrack_max=1000000
    sysctl -w net.netfilter.nf_conntrack_tcp_timeout_time_wait=30

TIME-WAIT Bucket Pool, Recycling and Reuse:

    net.ipv4.tcp_max_tw_buckets=1048576

    # Note: Enabling following option is not recommended. It could cause connection reset under NAT.
    # net.ipv4.tcp_tw_recycle = 1
    # net.ipv4.tcp_tw_reuse = 1

Timeout for FIN-WAIT-2 Sockets:

    net.ipv4.tcp_fin_timeout = 15

## Erlang VM Tuning

Tuning and optimize the Erlang VM in etc/emq.conf file:

    ## Erlang Process Limit
    node.process_limit = 2097152

    ## Sets the maximum number of simultaneously existing ports for this system
    node.max_ports = 1048576

## EMQ X Broker

Tune the acceptor pool, max_clients limit and sockopts for TCP listener in etc/emqx.conf:

    ## TCP Listener
    mqtt.listener.tcp.external= 1883
    mqtt.listener.tcp.external.acceptors = 64
    mqtt.listener.tcp.external.max_clients = 1000000

## Client Machine

Tune the client machine to benchmark emqttd broker:

    sysctl -w net.ipv4.ip_local_port_range="500 65535"
    echo 1000000 > /proc/sys/fs/nr_open
    ulimit -n 100000

### mqtt-jmeter

Test tool for concurrent connections: [ https://github.com/emqtt/mqtt-jmeter ](https://github.com/emqtt/mqtt-jmeter)
