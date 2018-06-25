<!-- toc -->

# Transfer

Transfer is a service for transmitting data. It receives data reported by Agent, processes fragmentation according to hash rules and finally push these fragmented data to modules like Graph and Judge.

##Service Deployment
Service deployment includes configuration changes, starting the service, testing the service, stopping the service etc. Before this, you need to unzip the the installation package to the deployment directory of the service.

```
# Change the configuration (the meaning of each setting is as follows)
mv cfg.example.json cfg.json
vim cfg.json

# Start the service
./open-falcon start transfer

# Check the service (provided service open the 6060 http monitor port, the result shows the service started correctly)
curl -s "127.0.0.1:6060/health"

# Stop the service
./open-falcon stop transfer

# Check the log
./open-falcon monitor transfer
```


## Deployment Information
The configuration file is "./cfg.json" and there will be an example configuration file "cfg.example.json" in the installation pack. The meaning of each item in configuration is as follow.

```
    debug: true/false, log will print debug information if it is true

    minStep: 30, allowed minimum time interval of data report, which is 30 seconds by default

    http
        - enabled: true/false, shows whether should open this http port; this port is a console port for sending control command, statistical command, debug command and so on
        - listen: http port that is being monitored 

    rpc
        - enabled: true/false, whether this jsonrpc data receiving port should be opened, through which Agent sends data
        - listen: http port that is being monitored

    socket #skip this setting as it is going to be neglected
        - enabled: true/false, whether should open this data receiving port in talent mode, which for user will facilitate sending data in lines to Transer 
        - listen: http port that is being monitored

    judge
        - enabled: true/false, whether is is able to send data to Judge
        - batch: batch size in data transfer that influences transfer speed, which is recommended to remain default
        - connTimeout: time-out setting of connection with the back-end measured in millisecond that can be slightly changed according to network quality, which is recommended to remain default
        - callTimeout: time-out setting of sending data to the back-end measured in millisecond that can be slightly changed according to network quality, which is recommended to remain default
        - pingMethod: ping port provided by the back-end to detect the connection is available or not, which must remain default
        - maxConns: maximum number of connections related to connection pool, which is recommended to remain default
        - maxIdle: maximum number of idle connections related to connection pool, which is recommended to remain default
        - replicas: number of node replicas that consistent hashing algorithm needs, which is recommended to remain default
        - cluster: stands for Judge list in the back-end in form of "key-value", in which "key" is the name of Judge in the back-end and "value" is specific ip:port

    graph
        - enabled: true/false, whether is is able to send data to Graph
        - batch: batch size in data transfer that influences transfer speed, which is recommended to remain default
        - connTimeout: time-out setting of connection with the back-end measured in millisecond that can be slightly changed according to network quality, which is recommended to remain default
        - callTimeout: time-out setting of sending data to the back-end measured in millisecond that can be slightly changed according to network quality, which is recommended to remain default
        - pingMethod: ping port provided by the back-end to detect the connection is available or not, which must remain default
        - maxConns: maximum number of connections related to connection pool, which is recommended to remain default
        - maxIdle: maximum number of idle connections related to connection pool, which is recommended to remain default
        - replicas: number of node replicas that consistent hashing algorithm needs, which is recommended to remain default
        - cluster: stands for Judge list in the back-end in form of "key-value", in which "key" is the name of Judge in the back-end and "value" is specific ip:port (if there are more than one address, they should be separated by comma; Transfer will send the same copy of data to each address, so multi-backup of data is based on this feature)

    tsdb
        - enabled: true/false, whether is is able to send data to open tsdb
        - batch: batch size in data transfer that influences transfer speed
        - connTimeout: time-out setting of connection with the back-end measured in millisecond that can be slightly changed according to network quality, which is recommended to remain default
        - callTimeout: time-out setting of sending data to the back-end measured in millisecond that can be slightly changed according to network quality, which is recommended to remain default
        - maxConns: maximum number of connections related to connection pool, which is recommended to remain default
        - maxIdle: maximum number of idle connections related to connection pool, which is recommended to remain default
        - retry: the number of retrying to connect with the back-end and retrying to send data
        - address: tsdb address or tsdb cluster VIP address, connecting tsbd through tcp
       
```

## Complementary Information
After Transfer module is deployed, please change the configuration of Agent, so that it points to the correct address of Transfer. After Graph and Judge are installed, please change the corresponding configuration of Transfer, so it can address Graph module and Judge module.


## Video Tutorial

We recorded a video tutorial for Transfer module on source-code-level: http://www.jikexueyuan.com/course/2061.html
