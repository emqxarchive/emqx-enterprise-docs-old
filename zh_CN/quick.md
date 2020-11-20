# 快速启动

假设部署两台 EMQ X Linux 节点集群，在云厂商 VPC 或私有网络内:

| 节点名             | IP 地址      |
| ------------------ | ------------ |
| emqx1@192.168.0.10 | 192.168.0.10 |
| emqx2@192.168.0.20 | 192.168.0.20 |

## 操作系统参数

EMQ X 在 Linux 环境下独立部署，支持 10 万线并发连接，需设置内核参数、TCP 协议栈参数。

### 系统全局文件句柄

系统全局允许分配的最大文件句柄数 256K:

    # 2 millions system-wide
    sysctl -w fs.file-max=262144
    sysctl -w fs.nr_open=262144
    echo 262144 > /proc/sys/fs/nr_open

允许当前会话/进程打开文件句柄数:

    ulimit -n 262144

### /etc/sysctl.conf

持久化'fs.file-max'设置到/etc/sysctl.conf 文件:

    fs.file-max = 262144

### /etc/security/limits.conf

/etc/security/limits.conf 持久化设置允许用户/进程打开文件句柄数:

    emqx      soft   nofile      262144
    emqx      hard   nofile      262144

注: Ubuntu 下需设置/etc/systemd/system.conf:

    DefaultLimitNOFILE=262144

## EMQ X 节点名称

设置节点名称与 Cookie(集群节点间通信认证)。

emqx1 节点/etc/emqx/emqx.conf 文件:

    node.name   = emqx1@192.168.0.10
    node.cookie = secret_dist_cookie

emqx2 节点/etc/emqx/emqx.conf 文件:

    node.name   = emqx2@192.168.0.20
    node.cookie = secret_dist_cookie

## EMQ X 节点启动

如果 RPM 或 DEB 方式安装，启动节点:

    service emqx start

如果独立 zip 包安装，启动节点:

    ./bin/emqx start

## EMQ X 节点集群

启动两台节点后，emqx1@192.168.0.10上执行:

    $ ./bin/emqx_ctl cluster join emqx2@192.168.0.20

    Join the cluster successfully.
    Cluster status: [{running_nodes,['emqx1@192.168.0.10','emqx2@192.168.0.20']}]

或，emqx2@192.168.0.20上执行:

    $ ./bin/emqx_ctl cluster join emqx1@192.168.0.10

    Join the cluster successfully.
    Cluster status: [{running_nodes,['emqx1@192.168.0.10','emqx2@192.168.0.20']}]

任意节点上查询集群状态:

    $ ./bin/emqx_ctl cluster status

    Cluster status: [{running_nodes,['emqx1@192.168.0.10','emqx2@192.168.0.20']}]

## Web 管理控制台

18083 端口是 Web 管理控制占用，该端口由'emqx-dashboard'插件启用。

控制台 URL: [ http:://localhost:18083/ ](http:://localhost:18083/) ，默认登录用户名: admin, 密码: public。

用户可以通过控制台，查询集群节点、MQTT 报文统计、MQTT 客户端、MQTT 会话与路由信息。

## MQTT 服务 TCP 端口

EMQ X 默认启用的外部 MQTT 服务端口包括:

| 1883  | MQTT 协议端口           |
| ----- | ----------------------- |
| 8883  | MQTT/SSL 端口           |
| 8083  | MQTT/WebSocket 端口     |
| 8084  | MQTT/WebSocket/SSL 端口 |
| 18083 | Web 管理控制台端口      |

上述占用端口可通过 etc/emqx.conf 配置文件的'Listeners'段落设置:

    ## External TCP Listener: 1883, 127.0.0.1:1883, ::1:1883
    listener.tcp.external = 0.0.0.0:1883

    ## SSL Listener: 8883, 127.0.0.1:8883, ::1:8883
    listener.ssl.external = 8883

    ## WebSocket Listener
    listener.ws.external = 8083

    ## External WSS Listener
    listener.wss.external = 8084

通过注释或删除相关段落，可禁用相关 TCP 服务启动。

## 节点集群 TCP 端口

EMQ X 节点间防火墙必须开放下述端口:

| 4369 | 集群节点发现端口 |
| ---- | ---------------- |
| 5369 | 集群节点数据通道 |
| 6369 | 集群节点控制通道 |
