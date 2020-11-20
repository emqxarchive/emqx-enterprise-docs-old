# Rate Limit

EMQ X Enterprise Edition supports rate limit in multiple aspects to ensure stability of the system.

## Max Cocurrent Connections

MQTT TCP or SSL lister, set the maximum concurrent connections:

    ## Maximum number of concurrent MQTT/TCP connections.
    ##
    ## Value: Number
    listener.tcp.\<name>.max_clients = 102400

    ## Maximum number of concurrent MQTT/SSL connections.
    ##
    ## Value: Number
    listener.ssl.\<name>.max_clients = 102400

## Max Connection Rate

MQTT TCP or SSL listener, set the maximum allowed connecting speed. It is set to 1000 connenctions per seconde by default:

    ## Maximum external connections per second.
    ##
    ## Value: Number
    listener.tcp.\<name>.max_conn_rate = 1000

    ## Maximum MQTT/SSL connections per second.
    ##
    ## Value: Number
    listener.ssl.\<name>.max_conn_rate = 1000

## Traffic Rate Limit

MQTT TCP or SSL listener, set rate limit for a single connection:

    ## Rate limit for the external MQTT/TCP connections. Format is 'rate,burst'.
    ##
    ## Value: rate,burst
    ## Unit: Bps
    ## listener.tcp.\<name>.rate_limit = 1024,4096

    ## Rate limit for the external MQTT/SSL connections.
    ##
    ## Value: rate,burst
    ## Unit: Bps
    ## listener.ssl.\<name>.rate_limit = 1024,4096

## Publish Rate Limit

MQTT TCP or SSL listenern, set message publishing rate limit for a single connection:

    ## Maximum publish rate of MQTT messages.
    ##
    ## Value: Number,Seconds
    ## Default: 10 messages per minute
    ## listener.tcp.\<name>.max_publish_rate = 10,60

    ## Maximum publish rate of MQTT messages.
    ##
    ## See: listener.tcp.\<name>.max_publish_rate
    ##
    ## Value: Number,Seconds
    ## Default: 10 messages per minute
    ## listener.ssl.external.max_publish_rate = 10,60
