##Transfer

Transfer provides data transfer service. It receives data reported by Agent, fragments the data based on the Hash rule, and pushes the fragmented data to components such as Graph and Judge.

##Source Code Compilation

```
# update common lib
cd $GOPATH/src/github.com/open-falcon/common
git pull

# compile
cd $GOPATH/src/github.com/open-falcon/transfer
go get ./...
./control build
./control pack
```
A tar.gz package will be packed at the last step. We can deploy with this package.

##Service Deployment

Service deployment includes configuration modification, starting service, verifying service, stopping service, etc. Before service deployment, you need to unzip the installation package to the deployment directory of the service.

```
# Modify configuration. Please refer to the following Please refer to the following for meanings of configuration items.
mv cfg.example.json cfg.json
vim cfg.json

# Start service
./control start

# Verify service. Here it is assumed that the service enables the 6060 http listening port. The verification result is ok, indicating that the service has started properly.
curl -s "127.0.0.1:6060/health"

...
# Stop service.
./control stop
```

After the service started, you can view the running status of the service through logs. The address of the log file is ./var/app.log. You can view data about the internal status of the server through the debug script ./test/debug. For example, you can run bash ./test/debug to obtain the statistics about the internal status of the server.

##Configuration Instruction

The default configuration file is ./cfg.json. By default, the installation package includes an example configuration file cfg.example.json. The meanings of configuration items are described as follows.


###Configuration

```
    debug: true/false, If it is set to true, debug information will be printed in logs.

    http
        - enable: true/false, indicates whether to enable this http port. This port is a control port and is used to send control commands, statistics commands, debug commands, etc. to Transfer.
        - listen: indicates the http port to be listened on.

    rpc
        - enable: true/false, indicates whether to enable the jsonrpc data receiving port. Agent uses this port to send data.
        - listen: the http port to be listened on.

    socket #To be abandoned. Avoid using it.
        - enable: true/false, indicates whether to enable the telnet data receiving port. This is to allow users to send data to Transfer line by line.
        - listen: indicates the http port to be listened on.

    judge
        - enable: true/false,  indicates whether to enable sending data to Judge.
        - batch: batch size for data transfer. It can accelerate the sending speed. It is recommended to keep the default value.
        - connTimeout: imeout for setting up a connection with the back end, in milliseconds. It can be slightly adjusted based on the network quality. It is recommended to keep the default value.
        - callTimeout: timeout for sending data to the back end, in milliseconds. It can be slightly adjusted based on the network quality. It is recommended to keep the default value.
        - pingMethod: ping interface provided by the back end. It is used to detect whether the connection is available. It must keep the default value.
        - maxConns: maximum number of connections, which is connection pool related configuration. It is recommended to keep the default value.
        - maxIdle: maximum number of idle connections, which is connection pool related configuration. It is recommended to keep the default value.
        - replicas: number of node copies needed by the consistency Hash algorithm. It is recommended to keep the default value.
        - cluster: dictionary in the key-value form, indicating the back-end Judge list. key stands for the back-end Judge name end and value stands for the specific IP:port.

    graph
        - enable: true/false, indicates whether to enable sending data to Graph.
        - batch: batch size for data transfer. It can accelerate the sending speed. It is recommended to keep the default value.
        - connTimeout: timeout for setting up a connection with the back end, in milliseconds. It can be slightly adjusted based on the network quality. It is recommended to keep the default value.
        - callTimeout: timeout for sending data to the back end, in milliseconds. It can be slightly adjusted based on the network quality. It is recommended to keep the default value.
        - pingMethod: ping interface provided by the back end. It is used to detect whether the connection is available. It must keep the default value.
        - maxConns: maximum number of connections, which is connection pool related configuration. It is recommended to keep the default value.
        - maxIdle: maximum number of idle connections, which is connection pool related configuration. It is recommended to keep the default value.
        - replicas: umber of node copies needed by the consistency Hash algorithm. It is recommended to keep the default value.
        - cluster: dictionary in the key-value form, indicating the back-end Graph list. key stands for the back-end Graph name and value stands for the specific IP:port. (Multiple addresses are separated by commas. Transfer will send the same data to each address. This feature can realize multiple backup of data.)

    tsdb
        - enabled: true/false, ndicates whether to enable sending data to tsdb.
        - batch: batch size for data transfer. It can accelerate the sending speed.
        - connTimeout: timeout for setting up a connection with the back end, in milliseconds. It can be slightly adjusted based on the network quality. It is recommended to keep the default value.
        - callTimeout: timeout for sending data to the back end, in milliseconds. It can be slightly adjusted based on the network quality. It is recommended to keep the default value.
        - maxConns:maximum number of connections, which is connection pool related configuration. It is recommended to keep the default value.
        - maxIdle:maximum number of idle connections, which is connection pool related configuration. It is recommended to keep the default value.
        - retry:  number of retries to connect with the back end and number of retries to send data
        - address: address of tsdb or address of tsdb cluster vip, connecting with tsdb through tcp.
     ```
##Supplementary Note

After deploying the Transfer component, modify the Agent configuration so that it points to the correct Transfer address. After installing Graph and Judge, modify the corresponding configurations of Transfer so that it can correctly address the two components.

##Video Course
We recorded a video for the Transfer module to provide interpretation at the source-code level:
http://www.jikexueyuan.com/course/2061.html




